<template>
	<div id="app">
		<div class="container-fluid">
			<div class="row">
				<div class="col-4">
					<MapPanel />
				</div>
				<div class="col-8">
					<Map />
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import Map from './components/map'
import MapPanel from './components/mapPanel'
export default {
	name: 'App',
	components: {
		Map, MapPanel
	}
}
</script>

<style lang="scss">
	#map-container {
		height: 100vh;
		margin-right: 0;
	}
</style>
