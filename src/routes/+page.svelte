<script lang="ts">
	import { post, replaceAccessToken } from '$lib/api';
	import { TextInput, PasswordInput, Button } from 'carbon-components-svelte';
	import { isLoggedIn } from '../store/auth';

	let id: string;
	let password: string;

	interface LoginResponse {
		grantType: string;
		accessToken: string;
	}

	const onSubmit = async () => {
		const res = await post<LoginResponse>('/api/login', { id, password });
		replaceAccessToken(res.accessToken);
		isLoggedIn.set(true);
	};

	let isCurrentLoggedIn: boolean;

	isLoggedIn.subscribe((isLoggedIn) => {
		isCurrentLoggedIn = isLoggedIn;
	});
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Svelte demo app" />
</svelte:head>

<section>
	<h1>FRJNS Backoffice</h1>

	{#if isCurrentLoggedIn}
		<p>Already logged in</p>
	{:else}
		<form on:submit={onSubmit}>
			<TextInput bind:value={id} labelText="ID" placeholder="Enter the admin ID" />
			<PasswordInput bind:value={password} labelText="PW" placeholder="Enter the admin PW" />
			<Button type="submit">Submit</Button>
		</form>
	{/if}
</section>

<style>
</style>
