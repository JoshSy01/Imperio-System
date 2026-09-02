<script lang="ts">
	import { enhance } from '$app/forms';
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();

	let loggingOut = $state(false);
	let filterDate = $state('');
	let selectedRegistro = $state<typeof data.registros[0] | null>(null);
	let showModal = $state(false);

	// Iniciales del usuario
	const initials = $derived(
		data.user?.name
			?.split(' ')
			.slice(0, 2)
			.map((w) => w[0])
			.join('')
			.toUpperCase() ?? '?'
	);

	// Mapa de productos por ID para búsqueda rápida
	const productosMap = $derived(
		Object.fromEntries(data.productos.map((p) => [p.idProducto, p]))
	);

	// Mapa de tipos por ID
	const tiposMap = $derived(
		Object.fromEntries(data.tipos.map((t) => [t.idTipo, t]))
	);

	// Formatear fecha a "DD/MM/YYYY"
	function formatFecha(fecha: string): string {
		const d = new Date(fecha);
		const day = String(d.getDate()).padStart(2, '0');
		const month = String(d.getMonth() + 1).padStart(2, '0');
		const year = d.getFullYear();
		return `${day}/${month}/${year}`;
	}

	// Clave de fecha "YYYY-MM-DD" para agrupar
	function fechaKey(fecha: string): string {
		const d = new Date(fecha);
		const day = String(d.getDate()).padStart(2, '0');
		const month = String(d.getMonth() + 1).padStart(2, '0');
		const year = d.getFullYear();
		return `${year}-${month}-${day}`;
	}

	// Fecha legible para cabecera de grupo
	function formatFechaGrupo(key: string): string {
		const [year, month, day] = key.split('-');
		const meses = ['Enero','Febrero','Marzo','Abril','Mayo','Junio','Julio','Agosto','Septiembre','Octubre','Noviembre','Diciembre'];
		return `${parseInt(day)} de ${meses[parseInt(month) - 1]} de ${year}`;
	}

	function formatHora(fecha: string): string {
		const d = new Date(fecha);
		return d.toLocaleTimeString('es-ES', { hour: '2-digit', minute: '2-digit' });
	}

	function formatPrice(price: number | null | undefined): string {
		if (price === null || price === undefined) return '—';
		return new Intl.NumberFormat('en-US', {
			style: 'currency',
			currency: 'USD'
		}).format(price);
	}

	// Total de un registro
	function calcTotal(reg: typeof data.registros[0]): number {
		return reg.preciosProductos.reduce((acc, precio, i) => {
			return acc + precio * (reg.cantidadProductos[i] ?? 1);
		}, 0);
	}

	// Ganancia de un registro
	function calcGanancia(reg: typeof data.registros[0]): number {
		const total = calcTotal(reg);
		return total * ((reg.porcentajeGanancia ?? 0) / 100);
	}

	// Registros filtrados por fecha
	const registrosFiltrados = $derived(
		data.registros.filter((r) => {
			if (!filterDate) return true;
			return fechaKey(r.fecha) === filterDate;
		})
	);

	// Agrupar registros por fecha (descendente)
	const registrosAgrupados = $derived(() => {
		const grupos: Record<string, typeof data.registros> = {};
		for (const r of registrosFiltrados) {
			const key = fechaKey(r.fecha);
			if (!grupos[key]) grupos[key] = [];
			grupos[key].push(r);
		}
		// Ordenar las claves de forma descendente
		const keys = Object.keys(grupos).sort((a, b) => b.localeCompare(a));
		return keys.map((key) => ({ key, registros: grupos[key] }));
	});

	function openModal(reg: typeof data.registros[0]) {
		selectedRegistro = reg;
		showModal = true;
	}

	function closeModal() {
		showModal = false;
		selectedRegistro = null;
	}

	function handleKeydown(e: KeyboardEvent) {
		if (e.key === 'Escape') closeModal();
	}

	// Nombre del producto por ID
	function getNombreProducto(id: number): string {
		return productosMap[id]?.nombreProducto ?? `Producto #${id}`;
	}

	// Nombre del tipo
	function getNombreTipo(idTipo: number | null | undefined): string {
		if (!idTipo) return '—';
		return tiposMap[idTipo]?.nombreTipo ?? `Tipo #${idTipo}`;
	}

	// Cantidad total de items en un registro
	function totalItems(reg: typeof data.registros[0]): number {
		return reg.cantidadProductos.reduce((a, b) => a + Number(b), 0);
	}
</script>

<svelte:head>
	<title>Visualizar Registros · Imperio</title>
	<meta name="description" content="Visualiza los registros de ventas en Imperio." />
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous" />
	<link
		href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<svelte:window onkeydown={handleKeydown} />

<div class="dashboard">
	<!-- Fondo -->
	<div class="bg-orbs" aria-hidden="true">
		<span class="orb orb-1"></span>
		<span class="orb orb-2"></span>
		<span class="orb orb-3"></span>
	</div>

	<!-- Navbar -->
	<header class="navbar">
		<div class="navbar-brand">
			<div class="brand-icon">
				<a href="/">
					<img src="/src/images/logo.png" width="50" height="50" alt="Imperio" class="brand-logo" />
				</a>
			</div>
		</div>

		<nav class="navbar-breadcrumb">
			<a href="/" class="breadcrumb-link">Inicio</a>
			<span class="breadcrumb-sep">›</span>
			<span class="breadcrumb-current">Visualizar Registros</span>
		</nav>

		<div class="navbar-user">
			<div class="user-avatar" aria-label="Avatar de {data.user?.name}">
				{initials}
			</div>
			<form method="POST" action="?/logout" use:enhance={() => {
				loggingOut = true;
				return async ({ update }) => { loggingOut = false; await update(); };
			}}>
				<button id="logout-btn" type="submit" class="logout-btn" disabled={loggingOut}>
					{#if loggingOut}
						<span class="spinner-sm" aria-hidden="true"></span>
					{:else}
						<svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
							<path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/>
							<polyline points="16 17 21 12 16 7"/>
							<line x1="21" y1="12" x2="9" y2="12"/>
						</svg>
					{/if}
					Cerrar sesión
				</button>
			</form>
		</div>
	</header>

	<!-- Contenido principal -->
	<main class="main-content">
		<!-- Page Header -->
		<div class="page-header">
			<div class="page-header-left">
				<div class="page-icon">📋</div>
				<div>
					<h1 class="page-title">Registros de Ventas</h1>
					<p class="page-subtitle">
						{data.registros.length} registro{data.registros.length !== 1 ? 's' : ''} en total
					</p>
				</div>
			</div>
		</div>

		<!-- Filtro por fecha -->
		<div class="filters-bar">
			<div class="filter-date-wrapper">
				<svg class="filter-icon" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
					<rect x="3" y="4" width="18" height="18" rx="2" ry="2"/>
					<line x1="16" y1="2" x2="16" y2="6"/>
					<line x1="8" y1="2" x2="8" y2="6"/>
					<line x1="3" y1="10" x2="21" y2="10"/>
				</svg>
				<input
					id="filter-date"
					type="date"
					class="date-input"
					bind:value={filterDate}
					aria-label="Filtrar por fecha"
				/>
				{#if filterDate}
					<button class="clear-filter" onclick={() => filterDate = ''} aria-label="Limpiar filtro">
						<svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
							<line x1="18" y1="6" x2="6" y2="18"/>
							<line x1="6" y1="6" x2="18" y2="18"/>
						</svg>
					</button>
				{/if}
			</div>
			{#if filterDate}
				<span class="filter-badge">
					Mostrando: {formatFechaGrupo(filterDate)}
				</span>
			{/if}
		</div>

		<!-- Contenido de registros -->
		{#if registrosFiltrados.length === 0}
			<div class="empty-state">
				<div class="empty-icon">📭</div>
				<h2>Sin registros</h2>
				<p>
					{filterDate
						? 'No se encontraron registros para esa fecha.'
						: 'Aún no hay registros realizados.'}
				</p>
			</div>
		{:else}
			{#each registrosAgrupados() as grupo (grupo.key)}
				<!-- Grupo por fecha -->
				<div class="date-group">
					<div class="date-group-header">
						<div class="date-group-line"></div>
						<span class="date-group-label">
							<svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
								<rect x="3" y="4" width="18" height="18" rx="2" ry="2"/>
								<line x1="16" y1="2" x2="16" y2="6"/>
								<line x1="8" y1="2" x2="8" y2="6"/>
								<line x1="3" y1="10" x2="21" y2="10"/>
							</svg>
							{formatFechaGrupo(grupo.key)}
						</span>
						<div class="date-group-line"></div>
					</div>

					<!-- Grid de registros del grupo -->
					<div class="registros-grid">
						{#each grupo.registros as reg (reg.idRegistro)}
							<div class="registro-card">
								<div class="registro-card-top">
									<div class="registro-meta">
										<span class="registro-id"># {reg.idRegistro}</span>
										<span class="registro-hora">
											<svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
												<circle cx="12" cy="12" r="10"/>
												<polyline points="12 6 12 12 16 14"/>
											</svg>
											{formatHora(reg.fecha)}
										</span>
									</div>
									<span class="registro-tipo-badge">
										{getNombreTipo(reg.idTipo)}
									</span>
								</div>

								<div class="registro-card-body">
									<div class="registro-stat">
										<span class="stat-label">Productos</span>
										<span class="stat-value">{reg.idProductos.length}</span>
									</div>
									<div class="registro-stat">
										<span class="stat-label">Unidades</span>
										<span class="stat-value">{totalItems(reg)}</span>
									</div>
									<div class="registro-stat">
										<span class="stat-label">Total</span>
										<span class="stat-value stat-total">{formatPrice(calcTotal(reg))}</span>
									</div>
									{#if (reg.porcentajeGanancia ?? 0) > 0}
										<div class="registro-stat">
											<span class="stat-label">Ganancia ({reg.porcentajeGanancia}%)</span>
											<span class="stat-value stat-ganancia">+{formatPrice(calcGanancia(reg))}</span>
										</div>
									{/if}
								</div>

								<!-- Productos resumidos -->
								<div class="registro-products-preview">
									{#each reg.idProductos.slice(0, 3) as pid, i}
										<span class="product-chip">
											{getNombreProducto(Number(pid))}
											{#if reg.cantidadProductos[i] > 1}
												<span class="product-chip-qty">×{reg.cantidadProductos[i]}</span>
											{/if}
										</span>
									{/each}
									{#if reg.idProductos.length > 3}
										<span class="product-chip product-chip-more">
											+{reg.idProductos.length - 3} más
										</span>
									{/if}
								</div>

								<button
									class="btn-ver-mas"
									onclick={() => openModal(reg)}
									aria-label="Ver detalles del registro {reg.idRegistro}"
								>
									<svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
										<path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/>
										<circle cx="12" cy="12" r="3"/>
									</svg>
									Ver más
								</button>
							</div>
						{/each}
					</div>
				</div>
			{/each}
		{/if}
	</main>
</div>

<!-- Modal de detalle del registro -->
{#if showModal && selectedRegistro}
	<!-- svelte-ignore a11y_no_noninteractive_element_interactions -->
	<div
		class="modal-backdrop"
		onclick={(e) => { if (e.target === e.currentTarget) closeModal(); }}
		onkeydown={(e) => { if (e.key === 'Escape') closeModal(); }}
		role="dialog"
		aria-modal="true"
		aria-label="Detalle del registro"
		tabindex="-1"
	>
		<div class="modal">
			<div class="modal-header">
				<div class="modal-header-info">
					<h2 class="modal-title">Registro #{selectedRegistro.idRegistro}</h2>
					<span class="modal-fecha">{formatFecha(selectedRegistro.fecha)} · {formatHora(selectedRegistro.fecha)}</span>
				</div>
				<button type="button" class="modal-close" onclick={closeModal} aria-label="Cerrar">
					<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
						<line x1="18" y1="6" x2="6" y2="18"/>
						<line x1="6" y1="6" x2="18" y2="18"/>
					</svg>
				</button>
			</div>

			<div class="modal-body">
				<!-- Info general -->
				<div class="modal-info-row">
					<div class="modal-info-item">
						<span class="modal-info-label">Tipo</span>
						<span class="modal-info-value">{getNombreTipo(selectedRegistro.idTipo)}</span>
					</div>
					<div class="modal-info-item">
						<span class="modal-info-label">Estado</span>
						<span class="modal-info-value estado-badge" class:activo={selectedRegistro.estado}>
							{selectedRegistro.estado ? 'Activo' : 'Inactivo'}
						</span>
					</div>
					<div class="modal-info-item">
						<span class="modal-info-label">Porcentaje de Ganancia</span>
						<span class="modal-info-value">{selectedRegistro.porcentajeGanancia ?? 0}%</span>
					</div>
				</div>

				<!-- Tabla de productos -->
				<div class="modal-section-title">Detalle de Productos</div>
				<div class="modal-table-wrap">
					<table class="modal-table">
						<thead>
							<tr>
								<th>Producto</th>
								<th class="th-center">Cantidad</th>
								<th class="th-right">Precio Unit.</th>
								<th class="th-right">Subtotal</th>
							</tr>
						</thead>
						<tbody>
							{#each selectedRegistro.idProductos as pid, i}
								{@const precio = selectedRegistro.preciosProductos[i] ?? 0}
								{@const cantidad = selectedRegistro.cantidadProductos[i] ?? 1}
								{@const subtotal = precio * Number(cantidad)}
								<tr>
									<td>{getNombreProducto(Number(pid))}</td>
									<td class="td-center">{cantidad}</td>
									<td class="td-right">{formatPrice(precio)}</td>
									<td class="td-right">{formatPrice(subtotal)}</td>
								</tr>
							{/each}
						</tbody>
					</table>
				</div>

				<!-- Totales -->
				<div class="modal-totals">
					<div class="modal-total-row">
						<span class="modal-total-label">Subtotal</span>
						<span class="modal-total-value">{formatPrice(calcTotal(selectedRegistro))}</span>
					</div>
					{#if (selectedRegistro.porcentajeGanancia ?? 0) > 0}
						<div class="modal-total-row">
							<span class="modal-total-label">Ganancia ({selectedRegistro.porcentajeGanancia}%)</span>
							<span class="modal-total-value ganancia-value">+{formatPrice(calcGanancia(selectedRegistro))}</span>
						</div>
						<div class="modal-total-row modal-total-final">
							<span class="modal-total-label">Total con Ganancia</span>
							<span class="modal-total-value">{formatPrice(calcTotal(selectedRegistro) + calcGanancia(selectedRegistro))}</span>
						</div>
					{/if}
				</div>
			</div>

			<div class="modal-footer">
				<button type="button" class="btn-cancel" onclick={closeModal}>Cerrar</button>
			</div>
		</div>
	</div>
{/if}

<style>
	*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

	.dashboard {
		min-height: 100vh;
		background: #080b14;
		font-family: 'Inter', system-ui, sans-serif;
		position: relative;
		overflow-x: hidden;
		color: #f1f5f9;
	}

	/* ── Orbs ── */
	.bg-orbs { position: fixed; inset: 0; pointer-events: none; z-index: 0; }
	.orb {
		position: absolute; border-radius: 50%;
		filter: blur(100px); opacity: 0.18;
		animation: drift 15s ease-in-out infinite;
	}
	.orb-1 {
		width: 600px; height: 600px;
		background: radial-gradient(circle, #6c3fff 0%, transparent 70%);
		top: -200px; left: -100px;
	}
	.orb-2 {
		width: 500px; height: 500px;
		background: radial-gradient(circle, #a855f7 0%, transparent 70%);
		bottom: -150px; right: -80px;
		animation-delay: -7s;
	}
	.orb-3 {
		width: 400px; height: 400px;
		background: radial-gradient(circle, #b6914c 0%, transparent 70%);
		top: 40%; left: 50%;
		animation-delay: -3s; opacity: 0.1;
	}
	@keyframes drift {
		0%, 100% { transform: scale(1) translate(0,0); }
		50%       { transform: scale(1.06) translate(20px, -15px); }
	}

	/* ── Navbar ── */
	.navbar {
		position: sticky; top: 0; z-index: 10;
		display: flex; align-items: center; justify-content: space-between;
		padding: 0.875rem 2rem;
		background: rgba(8, 11, 20, 0.85);
		border-bottom: 1px solid rgba(255,255,255,0.07);
		backdrop-filter: blur(16px);
		-webkit-backdrop-filter: blur(16px);
	}
	.navbar-brand { display: flex; align-items: center; gap: 0.625rem; }
	.brand-icon {
		width: 36px; height: 36px; border-radius: 10px;
		background: linear-gradient(135deg, #6c3fff, #a855f7);
		display: flex; align-items: center; justify-content: center;
		box-shadow: 0 2px 12px rgba(108,63,255,0.4);
	}
	.navbar-user { display: flex; align-items: center; gap: 0.875rem; }
	.user-avatar {
		width: 36px; height: 36px; border-radius: 50%;
		background: linear-gradient(135deg, #b6914c, #493d27);
		display: flex; align-items: center; justify-content: center;
		font-size: 0.8rem; font-weight: 700; color: white;
		box-shadow: 0 2px 12px rgba(73, 79, 27, 0.4);
		flex-shrink: 0;
	}
	.logout-btn {
		display: flex; align-items: center; gap: 0.4rem;
		background: rgba(255,255,255,0.06);
		border: 1px solid rgba(255,255,255,0.1);
		border-radius: 8px; padding: 0.45rem 0.85rem;
		color: #94a3b8; font-size: 0.8rem; font-family: inherit;
		cursor: pointer; transition: all 0.2s;
	}
	.logout-btn:hover:not(:disabled) {
		background: rgba(239,68,68,0.12);
		border-color: rgba(239,68,68,0.3);
		color: #fca5a5;
	}
	.logout-btn:disabled { opacity: 0.5; cursor: not-allowed; }
	.spinner-sm {
		width: 12px; height: 12px;
		border: 2px solid rgba(255,255,255,0.2);
		border-top-color: #94a3b8;
		border-radius: 50%;
		animation: spin 0.7s linear infinite;
		display: inline-block;
	}
	@keyframes spin { to { transform: rotate(360deg); } }

	/* ── Breadcrumb ── */
	.navbar-breadcrumb {
		display: flex; align-items: center; gap: 0.5rem;
		font-size: 0.9rem; color: #cbd5e1;
		font-family: inherit; font-weight: 500;
	}
	.breadcrumb-link { color: #94a3b8; text-decoration: none; transition: color 0.15s; }
	.breadcrumb-link:hover { color: #fff; }
	.breadcrumb-sep { color: #475569; }
	.breadcrumb-current { color: #fff; font-weight: 600; }

	/* ── Main ── */
	.main-content {
		position: relative; z-index: 1;
		max-width: 1100px; margin: 0 auto;
		padding: 2.5rem 1.5rem;
	}

	/* ── Page Header ── */
	.page-header {
		display: flex; align-items: center; justify-content: space-between;
		margin-bottom: 2rem;
		animation: slideUp 0.5s cubic-bezier(0.16,1,0.3,1) both;
	}
	.page-header-left { display: flex; align-items: center; gap: 1rem; }
	.page-icon {
		width: 52px; height: 52px; border-radius: 14px;
		background: linear-gradient(135deg, rgba(108,63,255,0.25), rgba(168,85,247,0.15));
		border: 1px solid rgba(108,63,255,0.3);
		display: flex; align-items: center; justify-content: center;
		font-size: 1.5rem;
		box-shadow: 0 4px 20px rgba(108,63,255,0.2);
	}
	.page-title {
		font-size: 1.6rem; font-weight: 800;
		letter-spacing: -0.03em;
		background: linear-gradient(135deg, #fff 40%, #a78bfa);
		-webkit-background-clip: text; -webkit-text-fill-color: transparent;
		background-clip: text;
	}
	.page-subtitle { font-size: 0.875rem; color: #64748b; margin-top: 0.2rem; }

	/* ── Filters ── */
	.filters-bar {
		display: flex; align-items: center; gap: 1rem;
		margin-bottom: 2rem;
		padding: 1rem 1.25rem;
		background: rgba(255,255,255,0.03);
		border: 1px solid rgba(255,255,255,0.07);
		border-radius: 14px;
		animation: slideUp 0.5s 0.05s cubic-bezier(0.16,1,0.3,1) both;
		flex-wrap: wrap;
	}
	.filter-date-wrapper {
		display: flex; align-items: center; gap: 0.5rem;
		background: rgba(255,255,255,0.05);
		border: 1px solid rgba(255,255,255,0.1);
		border-radius: 10px;
		padding: 0.45rem 0.75rem;
		transition: border-color 0.2s;
	}
	.filter-date-wrapper:focus-within {
		border-color: rgba(108,63,255,0.5);
		box-shadow: 0 0 0 3px rgba(108,63,255,0.12);
	}
	.filter-icon { color: #64748b; flex-shrink: 0; }
	.date-input {
		background: transparent; border: none; outline: none;
		color: #e2e8f0; font-family: inherit; font-size: 0.875rem;
		cursor: pointer;
	}
	.date-input::-webkit-calendar-picker-indicator {
		filter: invert(0.6); cursor: pointer;
	}
	.clear-filter {
		background: none; border: none; cursor: pointer;
		color: #64748b; display: flex; align-items: center;
		padding: 0; transition: color 0.2s;
	}
	.clear-filter:hover { color: #fca5a5; }
	.filter-badge {
		font-size: 0.78rem; font-weight: 600;
		color: #a78bfa;
		background: rgba(108,63,255,0.15);
		border: 1px solid rgba(108,63,255,0.25);
		border-radius: 20px; padding: 0.25rem 0.75rem;
	}

	/* ── Empty State ── */
	.empty-state {
		display: flex; flex-direction: column; align-items: center;
		gap: 0.75rem; padding: 4rem 2rem; text-align: center;
		background: rgba(255,255,255,0.02);
		border: 1px dashed rgba(255,255,255,0.08);
		border-radius: 20px;
	}
	.empty-icon { font-size: 3rem; }
	.empty-state h2 { font-size: 1.2rem; color: #e2e8f0; }
	.empty-state p { font-size: 0.875rem; color: #64748b; }

	/* ── Date Group ── */
	.date-group { margin-bottom: 2.5rem; animation: slideUp 0.5s 0.1s cubic-bezier(0.16,1,0.3,1) both; }
	.date-group-header {
		display: flex; align-items: center; gap: 1rem;
		margin-bottom: 1.25rem;
	}
	.date-group-line {
		flex: 1; height: 1px;
		background: linear-gradient(90deg, transparent, rgba(108,63,255,0.3), transparent);
	}
	.date-group-label {
		display: flex; align-items: center; gap: 0.5rem;
		font-size: 0.8rem; font-weight: 700;
		color: #a78bfa; letter-spacing: 0.04em;
		text-transform: uppercase; white-space: nowrap;
		background: rgba(108,63,255,0.1);
		border: 1px solid rgba(108,63,255,0.2);
		border-radius: 20px; padding: 0.3rem 0.9rem;
	}

	/* ── Registros Grid ── */
	.registros-grid {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
		gap: 1rem;
	}

	/* ── Registro Card ── */
	.registro-card {
		display: flex; flex-direction: column; gap: 1rem;
		background: rgba(255,255,255,0.03);
		border: 1px solid rgba(255,255,255,0.07);
		border-radius: 16px; padding: 1.25rem;
		transition: border-color 0.2s, background 0.2s, transform 0.2s, box-shadow 0.2s;
	}
	.registro-card:hover {
		border-color: rgba(108,63,255,0.3);
		background: rgba(108,63,255,0.05);
		transform: translateY(-2px);
		box-shadow: 0 8px 30px rgba(108,63,255,0.1);
	}

	.registro-card-top {
		display: flex; align-items: center; justify-content: space-between;
	}
	.registro-meta { display: flex; align-items: center; gap: 0.75rem; }
	.registro-id {
		font-size: 0.8rem; font-weight: 700; color: #64748b;
		font-family: 'SF Mono', 'Fira Code', monospace;
	}
	.registro-hora {
		display: flex; align-items: center; gap: 0.3rem;
		font-size: 0.75rem; color: #94a3b8;
	}
	.registro-tipo-badge {
		font-size: 0.72rem; font-weight: 600;
		color: #fbbf24;
		background: rgba(251,191,36,0.1);
		border: 1px solid rgba(251,191,36,0.2);
		border-radius: 20px; padding: 0.2rem 0.65rem;
		text-transform: capitalize;
	}

	.registro-card-body {
		display: grid; grid-template-columns: repeat(2, 1fr);
		gap: 0.75rem;
	}
	.registro-stat {
		display: flex; flex-direction: column; gap: 0.2rem;
		background: rgba(255,255,255,0.03);
		border: 1px solid rgba(255,255,255,0.05);
		border-radius: 10px; padding: 0.6rem 0.8rem;
	}
	.stat-label {
		font-size: 0.68rem; font-weight: 600;
		color: #475569; text-transform: uppercase; letter-spacing: 0.05em;
	}
	.stat-value {
		font-size: 0.95rem; font-weight: 700; color: #e2e8f0;
	}
	.stat-total { color: #a78bfa; }
	.stat-ganancia { color: #34d399; }

	/* Chips de productos */
	.registro-products-preview {
		display: flex; flex-wrap: wrap; gap: 0.4rem;
	}
	.product-chip {
		display: inline-flex; align-items: center; gap: 0.25rem;
		font-size: 0.72rem; font-weight: 500; color: #94a3b8;
		background: rgba(255,255,255,0.05);
		border: 1px solid rgba(255,255,255,0.08);
		border-radius: 6px; padding: 0.2rem 0.55rem;
		white-space: nowrap; max-width: 150px;
		overflow: hidden; text-overflow: ellipsis;
	}
	.product-chip-qty {
		color: #6c3fff; font-weight: 700; font-size: 0.65rem;
	}
	.product-chip-more {
		color: #a78bfa;
		background: rgba(108,63,255,0.1);
		border-color: rgba(108,63,255,0.2);
	}

	/* Botón ver más */
	.btn-ver-mas {
		display: flex; align-items: center; justify-content: center; gap: 0.5rem;
		width: 100%; padding: 0.6rem;
		background: linear-gradient(135deg, rgba(108,63,255,0.15), rgba(168,85,247,0.1));
		border: 1px solid rgba(108,63,255,0.25);
		border-radius: 10px;
		color: #a78bfa; font-size: 0.82rem; font-weight: 600;
		font-family: inherit; cursor: pointer;
		transition: all 0.2s;
	}
	.btn-ver-mas:hover {
		background: linear-gradient(135deg, rgba(108,63,255,0.3), rgba(168,85,247,0.2));
		border-color: rgba(108,63,255,0.5);
		color: #fff;
		transform: translateY(-1px);
		box-shadow: 0 4px 20px rgba(108,63,255,0.25);
	}

	/* ── Modal ── */
	.modal-backdrop {
		position: fixed; inset: 0; z-index: 50;
		background: rgba(0,0,0,0.7);
		backdrop-filter: blur(6px);
		-webkit-backdrop-filter: blur(6px);
		display: flex; align-items: center; justify-content: center;
		padding: 1rem;
		animation: fadeIn 0.15s ease;
	}
	@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
	.modal {
		width: 100%; max-width: 640px; max-height: 90vh;
		background: #0f1220;
		border: 1px solid rgba(255,255,255,0.1);
		border-radius: 20px;
		overflow: hidden; display: flex; flex-direction: column;
		box-shadow: 0 25px 80px rgba(0,0,0,0.6);
		animation: modalIn 0.25s cubic-bezier(0.16,1,0.3,1) both;
	}
	@keyframes modalIn {
		from { opacity: 0; transform: scale(0.95) translateY(12px); }
		to   { opacity: 1; transform: scale(1) translateY(0); }
	}
	.modal-header {
		display: flex; align-items: flex-start; justify-content: space-between;
		padding: 1.5rem 1.75rem 1.25rem;
		border-bottom: 1px solid rgba(255,255,255,0.07);
		background: rgba(108,63,255,0.05);
	}
	.modal-header-info { display: flex; flex-direction: column; gap: 0.25rem; }
	.modal-title { font-size: 1.2rem; font-weight: 700; color: #f1f5f9; }
	.modal-fecha { font-size: 0.8rem; color: #64748b; }
	.modal-close {
		background: rgba(255,255,255,0.06);
		border: 1px solid rgba(255,255,255,0.1);
		border-radius: 8px; padding: 0.4rem;
		color: #64748b; cursor: pointer;
		display: flex; align-items: center;
		transition: all 0.2s; flex-shrink: 0;
	}
	.modal-close:hover { background: rgba(239,68,68,0.12); color: #fca5a5; border-color: rgba(239,68,68,0.3); }

	.modal-body {
		flex: 1; overflow-y: auto;
		padding: 1.5rem 1.75rem;
		display: flex; flex-direction: column; gap: 1.25rem;
	}
	.modal-body::-webkit-scrollbar { width: 4px; }
	.modal-body::-webkit-scrollbar-track { background: transparent; }
	.modal-body::-webkit-scrollbar-thumb { background: rgba(108,63,255,0.3); border-radius: 4px; }

	/* Info general en modal */
	.modal-info-row {
		display: flex; gap: 1rem; flex-wrap: wrap;
	}
	.modal-info-item {
		flex: 1; min-width: 120px;
		display: flex; flex-direction: column; gap: 0.25rem;
		background: rgba(255,255,255,0.03);
		border: 1px solid rgba(255,255,255,0.06);
		border-radius: 10px; padding: 0.75rem 1rem;
	}
	.modal-info-label {
		font-size: 0.68rem; font-weight: 600;
		color: #475569; text-transform: uppercase; letter-spacing: 0.05em;
	}
	.modal-info-value { font-size: 0.9rem; font-weight: 600; color: #e2e8f0; }
	.estado-badge {
		display: inline-block;
		padding: 0.15rem 0.5rem;
		border-radius: 6px;
		font-size: 0.78rem;
		background: rgba(239,68,68,0.15);
		color: #fca5a5;
		border: 1px solid rgba(239,68,68,0.25);
		width: fit-content;
	}
	.estado-badge.activo {
		background: rgba(52,211,153,0.12);
		color: #34d399;
		border-color: rgba(52,211,153,0.25);
	}

	.modal-section-title {
		font-size: 0.75rem; font-weight: 700;
		color: #64748b; text-transform: uppercase; letter-spacing: 0.07em;
	}

	/* Tabla del modal */
	.modal-table-wrap { overflow-x: auto; border-radius: 12px; }
	.modal-table {
		width: 100%; border-collapse: collapse;
		font-size: 0.875rem;
	}
	.modal-table thead { background: rgba(108,63,255,0.08); }
	.modal-table th {
		padding: 0.75rem 1rem;
		text-align: left; font-size: 0.72rem;
		font-weight: 600; color: #64748b;
		text-transform: uppercase; letter-spacing: 0.05em;
		border-bottom: 1px solid rgba(255,255,255,0.06);
	}
	.modal-table td {
		padding: 0.75rem 1rem;
		color: #cbd5e1;
		border-bottom: 1px solid rgba(255,255,255,0.04);
	}
	.modal-table tbody tr:last-child td { border-bottom: none; }
	.modal-table tbody tr:hover td { background: rgba(255,255,255,0.02); }
	.th-center, .td-center { text-align: center; }
	.th-right, .td-right { text-align: right; }

	/* Totales del modal */
	.modal-totals {
		background: rgba(255,255,255,0.02);
		border: 1px solid rgba(255,255,255,0.06);
		border-radius: 12px; padding: 1rem 1.25rem;
		display: flex; flex-direction: column; gap: 0.6rem;
	}
	.modal-total-row { display: flex; justify-content: space-between; align-items: center; }
	.modal-total-label { font-size: 0.875rem; color: #94a3b8; }
	.modal-total-value { font-size: 0.95rem; font-weight: 700; color: #e2e8f0; }
	.ganancia-value { color: #34d399; }
	.modal-total-final {
		padding-top: 0.6rem;
		border-top: 1px solid rgba(255,255,255,0.07);
		margin-top: 0.2rem;
	}
	.modal-total-final .modal-total-label { color: #f1f5f9; font-weight: 600; }
	.modal-total-final .modal-total-value { color: #a78bfa; font-size: 1.1rem; }

	/* Modal footer */
	.modal-footer {
		padding: 1rem 1.75rem;
		border-top: 1px solid rgba(255,255,255,0.07);
		display: flex; justify-content: flex-end;
	}
	.btn-cancel {
		padding: 0.6rem 1.5rem;
		background: rgba(255,255,255,0.06);
		border: 1px solid rgba(255,255,255,0.1);
		border-radius: 10px;
		color: #94a3b8; font-size: 0.875rem;
		font-family: inherit; font-weight: 600;
		cursor: pointer; transition: all 0.2s;
	}
	.btn-cancel:hover { background: rgba(255,255,255,0.1); color: #e2e8f0; }

	@keyframes slideUp {
		from { opacity: 0; transform: translateY(20px); }
		to   { opacity: 1; transform: translateY(0); }
	}
</style>
