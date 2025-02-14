<script lang="ts">
	import { xxHash32 } from 'js-xxhash';
	import type { Node } from '$lib/common/types';
	import { onMount } from 'svelte';
	import { RouteStore } from '$lib/Stores';
	import { dateToStr } from '$lib/common/funcs';
	import { getDrawerStore, type DrawerSettings } from '@skeletonlabs/skeleton';
	import CardTileContainer from '../CardTileContainer.svelte';
	import CardTileEntry from '../CardTileEntry.svelte';
	import OnlineNodeIndicator from '$lib/parts/OnlineNodeIndicator.svelte';
	import OnlineUserIndicator from '$lib/parts/OnlineUserIndicator.svelte';

	export let node: Node;
	let routeCount: number = 0;
	const drawerStore = getDrawerStore();

	$: drawerSettings = {
		id: 'nodeDrawer-' + node.id,
		position: 'right',
		width: 'w-10/12 md:w-9/12 lg:w-8/12 xl:w-6/12',
		padding: '',
		meta: node,
	} as DrawerSettings;

	$: color = (xxHash32(node.id + ':' + node.givenName, 0xbeefbabe) & 0xff_ff_ff)
		.toString(16)
		.padStart(6, '0');

	onMount(() => {
		const unsubRouteStore = RouteStore.subscribe((routes) => {
			routeCount = routes.filter((r) => (r.node ?? r.machine).id == node.id).length;
		});
		return () => {
			unsubRouteStore();
		};
	});
</script>

<CardTileContainer onclick={(_) => drawerStore.open(drawerSettings)}>
	<div class="flex justify-between items-center mb-4 mt-2">
		<div class="flex items-center">
			<OnlineNodeIndicator {node} />
			<span class="ml-2 text-lg font-semibold">ID: {node.id}</span>
		</div>
		<div class="flex items-center font-bold">
			{node.givenName}
		</div>
	</div>
	<CardTileEntry title="Created:">
		{dateToStr(node.createdAt)}
	</CardTileEntry>
	<CardTileEntry title="User:">
		<div class="flex flex-row gap-3 items-center">
			{node.user.name}
			<OnlineUserIndicator user={node.user} />
		</div>
	</CardTileEntry>
	<CardTileEntry title="Routes:">
		{routeCount}
	</CardTileEntry>
	<hr style="background-color: #{color}" class="w-full h-0.5 mx-auto my-4 border-0 rounded" />
</CardTileContainer>
