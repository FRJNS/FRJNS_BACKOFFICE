<script lang="ts">
	import { onMount } from 'svelte';
	import { isLoggedIn } from '../../store/auth';
	import { goto } from '$app/navigation';
	import { del, get, post } from '$lib/api';
	import { Button, DatePicker, DatePickerInput, Modal, TextInput } from 'carbon-components-svelte';
	import { mmddyyyyToYyyymmdd } from '$lib/utils/date';

	let newDetail: string;
	let newStartDate: string;
	let newEndDate: string;

	const onSubmit = async () => {
		const startDate = mmddyyyyToYyyymmdd(newStartDate);
		const endDate = mmddyyyyToYyyymmdd(newEndDate);

		await post('/admin', { detail: newDetail, startDate, endDate });
    await refreshNews();
	};

	let news: News[] = [];

	const refreshNews = async () => {
		const res = await get<News[]>('/admin');
		news = [...res];
	};

	onMount(async () => {
		isLoggedIn.subscribe((isLoggedIn) => {
			if (!isLoggedIn) {
				goto('/');
				return;
			}
		});

		refreshNews();
	});

	let deletingId: number | null = null;
	let deletingDetail: string | null = null;
	let open = deletingId !== null;

	const onClickDelete = (id: number) => {
		deletingId = id;
		deletingDetail = news.find((news) => news.id === id)?.detail ?? null;
		open = true;
	};

	const deleteNewsById = async (id: number | null) => {
		if (id === null) return;

		await del(`/admin/${id}`);
		await refreshNews();
		open = false;
	};
</script>

<form on:submit={onSubmit}>
	<TextInput bind:value={newDetail} labelText="Detail" placeholder="Enter the Detail" />
	<DatePicker datePickerType="range" bind:valueFrom={newStartDate} bind:valueTo={newEndDate}>
		<DatePickerInput labelText="Start date" placeholder="mm/dd/yyyy" />
		<DatePickerInput labelText="End date" placeholder="mm/dd/yyyy" />
	</DatePicker>
	<Button type="submit">추가</Button>
</form>

<main>
	{#each news as eachNews}
		<div class="card">
			<h5>{eachNews.detail}</h5>
			<p>{eachNews.startDate} ~ {eachNews.endDate}</p>

			<Button on:click={() => onClickDelete(eachNews.id)} kind="danger-tertiary">삭제</Button>
		</div>
	{/each}

	<Modal
		bind:open
		modalHeading={`Delete news '${deletingDetail}'`}
		primaryButtonText="Confirm"
		secondaryButtonText="Cancel"
		on:click:button--primary={() => deleteNewsById(deletingId)}
		on:click:button--secondary={() => (open = false)}
		on:open
		on:close
		on:submit
	>
		<p>Are you sure you want to delete this news?</p>
	</Modal>
</main>

<style>
	form {
		padding-top: 30px;
		margin-bottom: 30px;
	}

	main {
		padding-top: 30px;
		width: 100%;

		display: flex;
		flex-direction: row;
		justify-content: center;
		flex-wrap: wrap;
		gap: 24px;
	}

	.card {
		width: 320px;
		display: flex;
		flex-direction: column;
		gap: 8px;
		padding: 12px 20px;
		border: 1px solid black;
	}
</style>
