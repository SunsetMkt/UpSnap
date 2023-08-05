<script lang="ts">
	import { goto } from '$app/navigation';
	import { pocketbase } from '$lib/stores/pocketbase';
	import { settingsPub } from '$lib/stores/settings';
	import { toggleVisibility } from '$lib/helpers/forms';
	import Fa from 'svelte-fa';
	import { faArrowRight, faEye } from '@fortawesome/free-solid-svg-icons';

	let stepsCompleted = 0;
	let inputPassword: HTMLInputElement;
	let inputConfirm: HTMLInputElement;
	let form = {
		email: '',
		password: '',
		confirm: ''
	};
	let errorMsg = '';

	async function register() {
		errorMsg = '';
		$pocketbase.admins
			.create({
				email: form.email,
				password: form.password,
				passwordConfirm: form.confirm
			})
			.then(() => {
				$pocketbase.admins
					.authWithPassword(form.email, form.password)
					.then(() => {
						stepsCompleted = 2;
					})
					.catch((err) => {
						if (err.data?.data?.identity?.message) {
							errorMsg = err.data.data.identity.message;
							return;
						}
					});
			})
			.catch((err) => {
				if (err.data?.data?.passwordConfirm?.message) {
					errorMsg = "Passwords don't match.";
				} else if (err.data?.data?.email?.message) {
					errorMsg = err.data.data.email.message;
				} else {
					errorMsg = err;
				}
			});
	}
</script>

<div class="flex h-screen">
	<div class="m-auto">
		<div class="flex flex-col gap-16 w-screen max-w-lg my-4">
			<div class="card bg-base-300 shadow-xl">
				{#if $settingsPub?.setup_completed}
					<figure class="w-72 mx-auto pt-6"><img src="/gopher.svg" alt="Gopher" /></figure>
					<div class="card-body">
						<h2 class="card-title">I didn't expect you here! 🧐</h2>
						<p>You are already done with the setup! Nothing to do.</p>
						<div class="card-actions justify-end">
							<button class="btn btn-primary" on:click={() => goto('/')}>Take me back</button>
						</div>
					</div>
				{:else if stepsCompleted === 0}
					<figure class="w-72 mx-auto pt-6"><img src="/gopher.svg" alt="Gopher" /></figure>
					<div class="card-body">
						<h2 class="card-title">Welcome to UpSnap 🥳</h2>
						<p>Please complete the following steps to finish the setup.</p>
						<div class="card-actions justify-end">
							<button class="btn btn-primary" on:click={() => (stepsCompleted = 1)}
								>Next <Fa icon={faArrowRight} /></button
							>
						</div>
					</div>
				{:else if stepsCompleted === 1}
					<div class="card-body">
						<div class="flex flex-row gap-4">
							<figure class="w-16"><img src="/gopher.svg" alt="Gopher" /></figure>
							<h2 class="card-title">Create an admin account</h2>
						</div>
						<form class="form-control w-full" on:submit|preventDefault={register}>
							<label class="label" for="email">
								<span class="label-text">Email:</span>
							</label>
							<input
								id="email"
								type="email"
								class="input input-bordered w-full"
								bind:value={form.email}
							/>
							<label class="label" for="password">
								<span class="label-text">Password:</span>
								<span class="label-text-alt">min. 10 characters</span>
							</label>
							<label class="relative block">
								<div
									class="absolute top-1/2 -translate-y-1/2 right-4 cursor-pointer"
									role="none"
									on:click={() => toggleVisibility(inputPassword)}
									on:keydown={() => toggleVisibility(inputPassword)}
								>
									<Fa icon={faEye} />
								</div>
								<input
									id="password"
									type="password"
									class="input input-bordered w-full"
									minlength="10"
									maxlength="72"
									bind:value={form.password}
									bind:this={inputPassword}
								/>
							</label>
							<label class="label" for="passwordConfirm">
								<span class="label-text">Password confirm:</span>
							</label>
							<label class="relative block">
								<div
									class="absolute top-1/2 -translate-y-1/2 right-4 cursor-pointer"
									role="none"
									on:click={() => toggleVisibility(inputConfirm)}
									on:keydown={() => toggleVisibility(inputConfirm)}
								>
									<Fa icon={faEye} />
								</div>
								<input
									id="confirm"
									type="password"
									class="input input-bordered w-full"
									minlength="10"
									maxlength="72"
									bind:value={form.confirm}
									bind:this={inputConfirm}
								/>
							</label>
							{#if errorMsg !== ''}
								<div class="alert alert-error mt-4">
									<svg
										xmlns="http://www.w3.org/2000/svg"
										class="stroke-current shrink-0 h-6 w-6"
										fill="none"
										viewBox="0 0 24 24"
										><path
											stroke-linecap="round"
											stroke-linejoin="round"
											stroke-width="2"
											d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z"
										/></svg
									>
									<span>{errorMsg}</span>
								</div>
							{/if}
							<div class="card-actions justify-end mt-4">
								<button class="btn btn-primary" type="submit"
									>Create <Fa icon={faArrowRight} /></button
								>
							</div>
						</form>
					</div>
				{:else if stepsCompleted === 2}
					<figure class="w-72 mx-auto pt-6"><img src="/gopher.svg" alt="Gopher" /></figure>
					<div class="card-body">
						<h2 class="card-title">You are all set! 🎉</h2>
						<p>Go ahead and add some devices to your dashboard.</p>
						<div class="card-actions justify-end">
							<button class="btn btn-success" on:click={() => goto('/')}>Lets go!</button>
						</div>
					</div>
				{/if}
			</div>
			{#if !$settingsPub?.setup_completed}
				<ul class="steps steps-horizontal">
					<li class="step step-primary">Welcome</li>
					<li class="step" class:step-primary={stepsCompleted > 0}>Create account</li>
					<li class="step" class:step-primary={stepsCompleted > 1}>Done</li>
				</ul>
			{/if}
		</div>
	</div>
</div>