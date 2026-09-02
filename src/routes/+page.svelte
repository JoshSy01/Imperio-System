<script lang="ts">
	import { enhance } from '$app/forms';
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();

	let loggingOut = $state(false);

	// Iniciales del usuario
	const initials = $derived(
		data.user?.name
			?.split(' ')
			.slice(0, 2)
			.map((w) => w[0])
			.join('')
			.toUpperCase() ?? '?'
	);
</script>

<svelte:head>
	<title>Dashboard · Imperio</title>
	<meta name="description" content="Tu panel de control en Imperio." />
</svelte:head>

<div class="dashboard">
	<!-- Fondo -->
	<div class="bg-orbs" aria-hidden="true">
		<span class="orb orb-1"></span>
		<span class="orb orb-2"></span>
	</div>

	<!-- Navbar -->
	<header class="navbar">
		<div class="navbar-brand">
			<div class="brand-icon">
				<img src="/src/images/logo.png" width="50" height="50" alt="Imperio" class="brand-logo">
			</div>
		</div>

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
		<!-- Welcome card -->
		<div class="welcome-card">
			<div class="welcome-avatar">{initials}</div>
			<div class="welcome-text">
				<h1>¡Bienvenido, {data.user?.name?.split(' ')[0]}! 👋</h1>
			</div>
		</div>

		<!-- Info cards -->
		<div class="info-grid">

				<div class="info-card">
					<img src="/src/images/yogurt.png" alt="Logo" class="info-logo">
					<a href="/gestion_yogurt" class="btn-primary">Gestión Registro Yogurt</a>
					<a href="/visualizar_yogurt" class="btn-secundary">Visualizar registros</a>
				</div>


			
				<div class="info-card">
					<img src="/src/images/proteinas.png" alt="Logo" class="info-logo">
					<a href="/gestion_proteinas" class="btn-primary">Gestión Registro Proteínas</a>
					<a href="/visualizar_proteinas" class="btn-secundary">Visualizar registros</a>
				</div>
			
		</div>
	</main>
</div>

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

	.bg-orbs { position: fixed; inset: 0; pointer-events: none; z-index: 0; }
	.orb {
		position: absolute; border-radius: 50%;
		filter: blur(100px); opacity: 0.2;
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
	@keyframes drift {
		0%, 100% { transform: scale(1) translate(0,0); }
		50%       { transform: scale(1.06) translate(20px, -15px); }
	}

	.navbar {
		position: sticky; top: 0; z-index: 10;
		display: flex; align-items: center; justify-content: space-between;
		padding: 0.875rem 2rem;
		background: rgba(8, 11, 20, 0.8);
		border-bottom: 1px solid rgba(255,255,255,0.07);
		backdrop-filter: blur(16px);
		-webkit-backdrop-filter: blur(16px);
	}
	.navbar-brand { display: flex; align-items: center; gap: 0.625rem; }
	.brand-icon {
		width: 36px; height: 36px; border-radius: 10px;
		background: linear-gradient(135deg, #6c3fff, #a855f7);
		display: flex; align-items: center; justify-content: center; color: white;
		box-shadow: 0 2px 12px rgba(108,63,255,0.4);
	}
	.navbar-name {
		font-size: 1.1rem; font-weight: 700; letter-spacing: -0.02em;
		background: linear-gradient(135deg, #fff 40%, #a78bfa);
		-webkit-background-clip: text; -webkit-text-fill-color: transparent;
		background-clip: text;
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
	}
	@keyframes spin { to { transform: rotate(360deg); } }

	.main-content {
		position: relative; z-index: 1;
		max-width: 900px; margin: 0 auto;
		padding: 2.5rem 1.5rem;
	}

	.welcome-card {
		display: flex; align-items: center; gap: 1.25rem;
		background: rgba(108, 63, 255, 0.08);
		border: 1px solid rgba(108, 63, 255, 0.2);
		border-radius: 20px; padding: 1.75rem 2rem;
		margin-bottom: 1.75rem;
		animation: slideUp 0.5s cubic-bezier(0.16,1,0.3,1) both;
	}
	.welcome-avatar {
		width: 60px; height: 60px; border-radius: 50%;
		background: linear-gradient(135deg, #b6914c, #493d27);
		display: flex; align-items: center; justify-content: center;
		font-size: 1.4rem; font-weight: 700; color: white;
		box-shadow: 0 4px 20px rgba(94, 87, 33, 0.5);
		flex-shrink: 0;
	}
	.welcome-text h1 {
		font-size: 1.5rem; font-weight: 700;
		letter-spacing: -0.02em; color: #f1f5f9;
		margin-bottom: 0.25rem;
	}
	.welcome-text p { font-size: 0.9rem; color: #64748b; }

	.info-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(300px, 5fr));
		gap: 4rem;
		animation: slideUp 0.5s 0.1s cubic-bezier(0.16,1,0.3,1) both;
		
	}
	.info-card {
		display: flex; align-items: center; gap: 1rem;
		background: rgba(255,255,255,0.04);
		border: 1px solid rgba(255,255,255,0.08);
		border-radius: 16px; padding: 1.25rem 1.4rem;
		transition: border-color 0.2s, background 0.2s;
		justify-content: center;
		flex-direction: column;
	}
	.info-card:hover {
		border-color: rgba(255,255,255,0.14);
		background: rgba(255,255,255,0.06);
	}
	.info-icon {
		width: 42px; height: 42px; border-radius: 10px;
		display: flex; align-items: center; justify-content: center;
		flex-shrink: 0;
	}
	.info-label {
		font-size: 0.72rem; font-weight: 600; color: #4b5563;
		text-transform: uppercase; letter-spacing: 0.06em;
		margin-bottom: 0.2rem;
	}
	.info-value {
		font-size: 0.9rem; font-weight: 500; color: #e2e8f0;
		white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
	}

	@keyframes slideUp {
		from { opacity: 0; transform: translateY(20px); }
		to   { opacity: 1; transform: translateY(0); }
	}

	/* ── Primary button ── */
	.btn-primary {
		width: 100%;
		margin-top: 0.5rem;
		padding: 0.875rem;
		border: none;
		border-radius: 12px;
		background: linear-gradient(135deg, #b6914c, #493d27);
		color: white;
		font-size: 0.975rem;
		font-weight: 600;
		font-family: inherit;
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 0.5rem;
		transition: transform 0.15s, box-shadow 0.15s, opacity 0.15s;
		box-shadow: 0 4px 24px rgba(179, 168, 88, 0.4);
		letter-spacing: 0.01em;
		text-decoration: none;
	}
	.btn-primary:hover:not(:disabled) {
		transform: translateY(-1px);
		box-shadow: 0 6px 30px rgba(216, 204, 127, 0.4);
	}
	.btn-primary:active:not(:disabled) { transform: translateY(0); }
	.btn-primary:disabled { opacity: 0.6; cursor: not-allowed; }

	.btn-secundary {
		width: 100%;
		margin-top: 0.5rem;
		padding: 0.875rem;
		border: none;
		border-radius: 12px;
		background: linear-gradient(135deg, rgb(44, 44, 44), #282828);
		color: white;
		font-size: 0.975rem;
		font-weight: 600;
		font-family: inherit;
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 0.5rem;
		transition: transform 0.15s, box-shadow 0.15s, opacity 0.15s;
		box-shadow: 0 4px 24px rgba(47, 47, 45, 0.4);
		letter-spacing: 0.01em;
		text-decoration: none;
	}
	.btn-secundary:hover:not(:disabled) {
		transform: translateY(-1px);
		box-shadow: 0 6px 30px rgba(47, 47, 45, 0.4);
	}
	.btn-secundary:active:not(:disabled) { transform: translateY(0); }
	.btn-secundary:disabled { opacity: 0.6; cursor: not-allowed; }
</style>
