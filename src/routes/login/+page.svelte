<script lang="ts">
	import { enhance } from '$app/forms';
	import type { ActionData } from './$types';

	let { form }: { form: ActionData } = $props();

	let loading = $state(false);
	let showPassword = $state(false);
</script>

<svelte:head>
	<title>Iniciar Sesión · Imperio</title>
	<meta name="description" content="Accede a tu cuenta en Imperio." />
</svelte:head>

<div class="login-wrapper">
	<!-- Fondo animado -->
	<div class="bg-orbs" aria-hidden="true">
		<span class="orb orb-1"></span>
		<span class="orb orb-2"></span>
		<span class="orb orb-3"></span>
	</div>

	<div class="login-card">
		<!-- Logo / Marca -->
		<div class="brand" style="justify-content: center;">
			<img src="https://res.cloudinary.com/b6k3zfto/image/upload/v1788471377/logo.png" width="75" height="75" alt="Imperio" class="brand-logo">
		</div>

		<div class="card-header">
			<h2>Bienvenido de vuelta Joel</h2>
			<p>Ingresa la contraseña para continuar</p>
		</div>

		{#if form?.error}
			<div class="error-alert" role="alert">
				<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
					<circle cx="12" cy="12" r="10"/>
					<line x1="12" y1="8" x2="12" y2="12"/>
					<line x1="12" y1="16" x2="12.01" y2="16"/>
				</svg>
				{form.error}
			</div>
		{/if}

		<form
			id="login-form"
			method="POST"
			use:enhance={() => {
				loading = true;
				return async ({ update }) => {
					loading = false;
					await update();
				};
			}}
		>

			<div class="field">
				<label for="password">Contraseña</label>
				<div class="input-wrapper">
					<svg class="input-icon" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
						<rect x="3" y="11" width="18" height="11" rx="2" ry="2"/>
						<path d="M7 11V7a5 5 0 0 1 10 0v4"/>
					</svg>
					<input
						id="password"
						name="password"
						type={showPassword ? 'text' : 'password'}
						placeholder="••••••••"
						autocomplete="current-password"
						required
					/>
					<button
						type="button"
						class="toggle-password"
						onclick={() => (showPassword = !showPassword)}
						aria-label={showPassword ? 'Ocultar contraseña' : 'Mostrar contraseña'}
					>
						{#if showPassword}
							<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
								<path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94"/>
								<path d="M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19"/>
								<line x1="1" y1="1" x2="23" y2="23"/>
							</svg>
						{:else}
							<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
								<path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/>
								<circle cx="12" cy="12" r="3"/>
							</svg>
						{/if}
					</button>
				</div>
			</div>

			<button id="login-submit" type="submit" class="btn-primary" disabled={loading}>
				{#if loading}
					<span class="spinner" aria-hidden="true"></span>
					Iniciando sesión...
				{:else}
					Iniciar sesión
				{/if}
			</button>
		</form>
	</div>
</div>

<style>
	/* ── Reset & base ── */
	*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

	/* ── Wrapper ── */
	.login-wrapper {
		min-height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
		background: #0e1018;
		position: relative;
		overflow: hidden;
		font-family: 'Inter', system-ui, sans-serif;
		padding: 1.5rem;
	}

	/* ── Orbs de fondo ── */
	.bg-orbs { position: absolute; inset: 0; pointer-events: none; }

	.orb {
		position: absolute;
		border-radius: 50%;
		filter: blur(80px);
		opacity: 0.35;
		animation: drift 12s ease-in-out infinite;
	}
	.orb-1 {
		width: 520px; height: 520px;
		background: radial-gradient(circle, #0f0c1a 0%, transparent 70%);
		top: -180px; left: -120px;
		animation-delay: 0s;
	}
	.orb-2 {
		width: 420px; height: 420px;
		background: radial-gradient(circle, #272829 0%, transparent 70%);
		bottom: -140px; right: -100px;
		animation-delay: -4s;
	}
	.orb-3 {
		width: 300px; height: 300px;
		background: radial-gradient(circle, #353238 0%, transparent 70%);
		top: 50%; left: 50%;
		transform: translate(-50%, -50%);
		animation-delay: -8s;
	}

	@keyframes drift {
		0%, 100% { transform: translate(0, 0) scale(1); }
		33%       { transform: translate(30px, -20px) scale(1.05); }
		66%       { transform: translate(-20px, 30px) scale(0.97); }
	}

	/* ── Card ── */
	.login-card {
		position: relative;
		width: 100%;
		max-width: 420px;
		background: rgba(38, 41, 36, 0.1);
		border: 1px solid rgba(255, 255, 255, 0.1);
		border-radius: 24px;
		padding: 2.5rem;
		backdrop-filter: blur(20px);
		-webkit-backdrop-filter: blur(20px);
		box-shadow:
			0 0 0 1px rgba(108, 63, 255, 0.15),
			0 25px 60px rgba(0, 0, 0, 0.5),
			inset 0 1px 0 rgba(255, 255, 255, 0.08);
		animation: slideUp 0.5s cubic-bezier(0.16, 1, 0.3, 1) both;
	}

	@keyframes slideUp {
		from { opacity: 0; transform: translateY(30px); }
		to   { opacity: 1; transform: translateY(0); }
	}

	/* ── Brand ── */
	.brand {
		display: flex;
		align-items: center;
		gap: 0.625rem;
		margin-bottom: 2rem;
	}
	.brand-icon {
		width: 44px; height: 44px;
		border-radius: 12px;
		display: flex; align-items: center; justify-content: center;
	}
	.brand-name {
		font-size: 1.35rem;
		font-weight: 700;
		letter-spacing: -0.02em;
		background: linear-gradient(135deg, #fff 40%, #a78bfa);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
	}

	/* ── Card header ── */
	.card-header {
		margin-bottom: 1.75rem;
	}
	.card-header h2 {
		font-size: 1.6rem;
		font-weight: 700;
		letter-spacing: -0.03em;
		color: #f1f5f9;
		margin-bottom: 0.35rem;
	}
	.card-header p {
		font-size: 0.9rem;
		color: #64748b;
	}

	/* ── Error alert ── */
	.error-alert {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		background: rgba(239, 68, 68, 0.12);
		border: 1px solid rgba(239, 68, 68, 0.3);
		border-radius: 10px;
		padding: 0.75rem 1rem;
		color: #fca5a5;
		font-size: 0.875rem;
		margin-bottom: 1.25rem;
		animation: shake 0.3s ease;
	}
	@keyframes shake {
		0%, 100% { transform: translateX(0); }
		25%       { transform: translateX(-4px); }
		75%       { transform: translateX(4px); }
	}

	/* ── Form fields ── */
	.field {
		margin-bottom: 1.1rem;
	}
	.field label {
		display: block;
		font-size: 0.8rem;
		font-weight: 600;
		color: #94a3b8;
		letter-spacing: 0.05em;
		text-transform: uppercase;
		margin-bottom: 0.45rem;
	}

	.input-wrapper {
		position: relative;
		display: flex;
		align-items: center;
	}
	.input-icon {
		position: absolute;
		left: 0.875rem;
		color: #4b5563;
		pointer-events: none;
		flex-shrink: 0;
	}
	.input-wrapper input {
		width: 100%;
		background: rgba(255, 255, 255, 0.05);
		border: 1px solid rgba(255, 255, 255, 0.08);
		border-radius: 12px;
		padding: 0.8rem 3rem 0.8rem 2.75rem;
		color: #f1f5f9;
		font-size: 0.95rem;
		font-family: inherit;
		outline: none;
		transition: border-color 0.2s, background 0.2s, box-shadow 0.2s;
	}
	.input-wrapper input::placeholder { color: #374151; }
	.input-wrapper input:focus {
		border-color: rgba(255, 221, 98, 0.4);
		background: rgba(216, 204, 127, 0.07);
		box-shadow: 0 0 0 3px rgba(255, 221, 98, 0.4);
	}
	.input-wrapper input:hover:not(:focus) {
		border-color: rgba(255, 255, 255, 0.15);
	}

	.toggle-password {
		position: absolute;
		right: 0.875rem;
		background: none;
		border: none;
		cursor: pointer;
		color: #4b5563;
		display: flex;
		align-items: center;
		padding: 0;
		transition: color 0.2s;
	}
	.toggle-password:hover { color: #9ca3af; }

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
	}
	.btn-primary:hover:not(:disabled) {
		transform: translateY(-1px);
		box-shadow: 0 6px 30px rgba(216, 204, 127, 0.4);
	}
	.btn-primary:active:not(:disabled) { transform: translateY(0); }
	.btn-primary:disabled { opacity: 0.6; cursor: not-allowed; }

	/* ── Spinner ── */
	.spinner {
		width: 16px; height: 16px;
		border: 2px solid rgba(255,255,255,0.3);
		border-top-color: white;
		border-radius: 50%;
		animation: spin 0.7s linear infinite;
	}
	@keyframes spin { to { transform: rotate(360deg); } }

	/* ── Switch link ── */
	.switch-link {
		margin-top: 1.5rem;
		text-align: center;
		font-size: 0.875rem;
		color: #4b5563;
	}
	.switch-link a {
		color: #a78bfa;
		text-decoration: none;
		font-weight: 600;
		transition: color 0.2s;
	}
	.switch-link a:hover { color: #c4b5fd; text-decoration: underline; }
</style>
