<template>
    <div class="map-container">
        <div class="map-info">
            <span>Zoom: {{currentZoom}}</span>
            <span>Center: {{currentCenter}}</span>
        </div>
        <l-map :zoom="zoom" :center="center" :options="mapOptions" @update:center="centerUpdate" @update:zoom="zoomUpdate">
            <l-tile-layer :url="url" />
            <l-circle-marker
                :lat-lng="point.position"
                :radius="point.size"
                :color="point.color"
                v-for="(point,i) in circles" :key="point.type + i">
                <l-popup>
                    <div>
                        <b>Назва:</b> <span>{{point.caption}}</span>
                    </div>
                    <div>
                        <b>Координати:</b> <span>{{point.position}}</span>
                    </div>
                </l-popup>
            </l-circle-marker>
            <l-rectangle 
                :bounds="point.bounds" 
                :l-style="{color: point.color, weight: point.weight}"
                v-for="(point,i) in rectangles" :key="point.type + i">
            </l-rectangle>
            <l-polygon 
                :lat-lngs="point.bounds" 
                :color="point.color"
                :weight="point.weight"
                fillColor="rgba(51,136,255,0.25)"
                v-for="(point,i) in polygons" :key="point.type + i">
            </l-polygon>
        </l-map>
    </div>
</template>

<script>
import { latLng } from "leaflet"
import { LMap, LTileLayer, LPopup, LCircleMarker, LRectangle, LPolygon } from "vue2-leaflet"
import kievCoord from '../kiev.coord'

export default {
    components: {
        LMap, LTileLayer, LPopup, LCircleMarker, LRectangle, LPolygon
    },
    data() {
        return {
            zoom: 8,
            center: latLng(50.33319, 30.42663),
            // url: 'https://tiles.stadiamaps.com/tiles/alidade_smooth_dark/{z}/{x}/{y}{r}.png',
            url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
            currentZoom: 8,
            currentCenter: latLng(50.33319, 30.42663),
            mapOptions: {zoomSnap: 0.5},
            points: [
                {
                    caption: 'Київська область',
                    type: 'Polygon',
                    size: 6,
                    weight: 2,
                    color: '#0380e7',
                    position: '',
                    bounds: kievCoord,
                },
                {
                    caption: 'SOFTPRO',
                    type: 'Circle',
                    size: 6,
                    weight: 2,
                    color: 'red',
                    position: [50.428911, 30.516425],
                    bounds: ''
                },
                {
                    caption: 'Обухів',
                    type: 'Rectangle',
                    size: 6,
                    weight: 2,
                    color: '#1a9a03',
                    position: '',
                    bounds: [[50.122779,30.58165], [50.086187,30.665163]]
                }
            ]
        }
    },
    computed: {
        circles() {
            return this.points.filter(point => point.type === 'Circle')
        },
        rectangles() {
            return this.points.filter(point => point.type === 'Rectangle')
        },
        polygons() {
            return this.points.filter(point => point.type === 'Polygon')
        }
    },
    methods: {
        centerUpdate(center) {
            this.currentCenter = center
            this.center = center
        },
        zoomUpdate(zoom) {
            this.currentZoom = zoom
        },
        pointCenter(point) {
            if (point.type === 'Circle') {
                this.center = [point.position[0], point.position[1]]
            }
            if (point.type === 'Rectangle') {
                let x = (point.bounds[0][0] + point.bounds[1][0]) / 2
                let y = (point.bounds[0][1] + point.bounds[1][1]) / 2
                this.center = [x,y]
            }
        },
        setPoints(data) {
            this.points = data
        }
    },
    mounted() {
        this.$eventBus.$on('setView', this.pointCenter)
        this.$eventBus.$on('setPoints', this.setPoints)
    }
}
</script>

<style lang="scss">

.map-container {
    height: 100vh;
    margin: 0 -15px;
    position: relative;
}
.map-info {
    height: 20px;
    padding: 0;
    font-size: 13px;
    display: inline-block;
    position: absolute;
    top: 0;
    right: 0;
    z-index: 999;
    background: #fff;
    span {
        display: inline-flex;
        align-items: center;
        justify-content: flex-start;
        padding: 0 10px;
        height: 100%;
        border-right: 1px solid #eee;
    }
}

</style>