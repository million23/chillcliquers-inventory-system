<script>
	import { onMount } from 'svelte';
	import { fly } from 'svelte/transition';
	import { supabase } from '../../../../_global';
	import Toastify from 'toastify-js';

	let items = [];
	let loaded = false;
	let selectedItem = '';

	onMount(async (e) => {
		getData();
		loaded = true;
	});

	const getData = async () => {
		loaded = false;
		let { data: inventory, error } = await supabase.from('inventory').select('*').eq('isArchived', 'true');
		if (!error) {
			items = [...inventory];
			loaded = true;
		}
	};

	const restoreItem = async (itemID) => {
		let { data: inventory, error } = await supabase.from('inventory').update({ isArchived: 'false' }).eq('id', itemID);
		if (!error) {
			items = [...inventory];
			getData();
			Toastify({
				text: 'Item Restored',
				gravity: 'bottom',
				position: 'right',
				backgroundColor: '#15C39A',
				style: {
					width: '300px',
					color: '#000'
				}
			}).showToast();
		} else {
			alert(JSON.stringify(error));
		}
	};

	const removeItem = async (itemID) => {
		let { data: inventory, error } = await supabase.from('inventory').delete().eq('id', itemID);
		if (!error) {
			items = [...inventory];
			getData();
			Toastify({
				text: 'Item Removed',
				gravity: 'bottom',
				position: 'right',
				backgroundColor: '#F0583C',
				style: {
					width: '300px',
					color: '#fff'
				}
			}).showToast();
		} else {
			alert(JSON.stringify(error));
		}
	};
</script>

<main class="mt-5" in:fly={{ y: 40, duration: 500 }}>
	<h3>Archive List</h3>
	{#if loaded}
		{#if items.length > 0}
			<table class="mt-5 text-start table" style="user-select: text;" in:fly|local={{ y: 40, duration: 500, delay: 200 }}>
				<thead>
					<tr>
						<th scope="col">Item Code</th>
						<th scope="col">Item Name</th>
						<th scope="col">Added By</th>
						<th scope="col">Item stored</th>
						<th scope="col">Item Category</th>
						<th scope="col">Item Type</th>
						<th scope="col" />
					</tr>
				</thead>
				<tbody>
					{#each items as item, index}
						<tr class="align-middle" in:fly={{ y: 20, duration: 500, delay: 100 + index * 50 }}>
							<td>{item.item_code}</td>
							<td>{item.item_name}</td>
							<td>{item.added_by.split('@')[0]}</td>
							<td>{item.item_count}</td>
							<td style="text-transform: uppercase;">{item.category}</td>

							<td>{item.item_count}</td>
							<td>
								<div class="btn-group" role="group">
									<button
										class="btn btn-outline-danger"
										on:click={(e) => {
											removeItem(item.id);
										}}><i class="bi bi-trash" /></button
									>
									<button class="btn btn-outline-primary"
										><i
											class="bi bi-arrow-clockwise"
											on:click={() => {
												restoreItem(item.id);
											}}
										/></button
									>
								</div>
							</td>
						</tr>
					{/each}
				</tbody>
			</table>
		{:else}
			<div in:fly|local={{ y: 40, duration: 500, delay: 200 }}>
				<p class="lead mt-5">Seems like its empty</p>
			</div>
		{/if}
	{/if}
	{#if !loaded}
		<div class="lds-ellipsis" in:fly={{ y: 40, duration: 500 }}>
			<div />
			<div />
			<div />
			<div />
		</div>
	{/if}
</main>

<style>
	.lds-ellipsis {
		display: inline-block;
		position: relative;
		width: 80px;
		height: 80px;
	}
	.lds-ellipsis div {
		position: absolute;
		top: 33px;
		width: 13px;
		height: 13px;
		border-radius: 50%;
		background: black;
		animation-timing-function: cubic-bezier(0, 1, 1, 0);
	}
	.lds-ellipsis div:nth-child(1) {
		left: 8px;
		animation: lds-ellipsis1 400ms infinite;
	}
	.lds-ellipsis div:nth-child(2) {
		left: 8px;
		animation: lds-ellipsis2 400ms infinite;
	}
	.lds-ellipsis div:nth-child(3) {
		left: 32px;
		animation: lds-ellipsis2 400ms infinite;
	}
	.lds-ellipsis div:nth-child(4) {
		left: 56px;
		animation: lds-ellipsis3 400ms infinite;
	}
	@keyframes lds-ellipsis1 {
		0% {
			transform: scale(0);
		}
		100% {
			transform: scale(1);
		}
	}
	@keyframes lds-ellipsis3 {
		0% {
			transform: scale(1);
		}
		100% {
			transform: scale(0);
		}
	}
	@keyframes lds-ellipsis2 {
		0% {
			transform: translate(0, 0);
		}
		100% {
			transform: translate(24px, 0);
		}
	}
</style>
