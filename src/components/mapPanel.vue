<template>
    <div class="map-panel">
        <div class="map-panel__top">
            <markerForm @submit="newPoint" />
        </div>
        <div class="map-panel__bottom">
            <ul class="map-panel__points-list">
                <li class="map-panel__point" v-for="(point,i) in points" :key="i">
                    <h3>{{point.caption}} <small :style="`color: ${point.color}`">({{point.type}})</small></h3>
                    <small v-if="point.type === 'Circle'">{{point.position[0]}}, {{point.position[1]}}</small>
                    <small v-else-if="point.type === 'Rectangle'">{{point.bounds[0].join(',')}}; {{point.bounds[1].join(',')}}</small>
                    <small v-else-if="point.type === 'Polygon'">Точок: {{point.bounds.length}}</small>
                    <div class="map-panel__point-actions">
                        <b-button class="mr-2" variant="outline-primary" size="sm" @click.prevent="setView(point)" v-if="point.type !== 'Polygon'" title="Відцентрувати">
                            <b-icon-map></b-icon-map>
                        </b-button>
                        <b-button class="mr-2" variant="outline-primary" size="sm" @click.prevent="saveCsv(point)" title="Завантажити .csv">
                            <b-icon-download></b-icon-download>
                        </b-button>
                        <b-button class="mr-2" variant="outline-primary" size="sm" @click.prevent="edit(point)" title="Редагувати">
                            <b-icon-pencil-square></b-icon-pencil-square>
                        </b-button>
                        <b-button class="mr-2" variant="danger" size="sm" @click.prevent="remove(point)" title="Видалити">
                            <b-icon-trash></b-icon-trash>
                        </b-button>
                    </div>
                </li>
            </ul>
        </div>
    </div>
</template>

<script>
import markerForm from './markerForm'
import kievCoord from '../kiev.coord'
import { saveAs } from 'file-saver'

export default {
    components: {
        markerForm
    },
    data() {
        return {
            points: [
                {
                    caption: 'Київська область',
                    type: 'Polygon',
                    bounds: kievCoord,
                    weight: 2,
                    color: '#0380e7'
                },
                {
                    caption: 'Київ',
                    type: 'Rectangle',
                    bounds: [[50.53198, 30.315397], [50.385975, 30.68001]],
                    weight: 2,
                    color: 'red'
                },
                {
                    caption: 'Обухів',
                    type: 'Circle',
                    position: [50.109621, 30.626304],
                    size: 6,
                    color: 'orange'
                }
            ],
        }
    },
    methods: {
        newPoint(data) {
            let bounds
            let newBounds = []
            if (data.bounds !== '') {
                if (data.bounds.includes('[' && ']')) {
                   newBounds = JSON.parse(data.bounds)
                } else {
                    bounds = data.bounds.trim().split(';')
                    bounds.forEach(bound => {
                        let boundArr = bound.split(',')
                        newBounds.push([+boundArr[0],+boundArr[1]])
                    })
                }
                
            } else {
                newBounds = ''
            }
            this.points.push({
                caption: data.caption,
                type: data.type,
                position: data.position.trim().split(','),
                size: data.size,
                weight: data.weight,
                color: data.color,
                bounds: newBounds
            })
            this.$eventBus.$emit('setPoints', this.points)
        },
        setView(point) {
            this.$eventBus.$emit('setView', point)
        },
        saveCsv(point) {
            let data = [[point.caption], [point.type], [point.size], [point.color], [point.position]]
            data = data.map(el => {return el[0] + "\r\n"})
            saveAs( new Blob( data, {type : 'data:text/csv;charset=utf-8;'}), `${point.caption}.csv`, { autoBom: true } )
        },
        edit(point) {

        },
        remove(point) {
            this.$bvModal.msgBoxConfirm('Видалити точку?', {okTitle: 'Так', cancelTitle: 'Ні', okVariant: 'danger'})
            .then(value => {
                if (value) {
                    this.points.splice(this.points.indexOf(point), 1)
                    this.$eventBus.$emit('setPoints', this.points)
                }
            })
        }
    },
    created() {
        this.$eventBus.$emit('setPoints', this.points)
    }
}
</script>

<style lang="scss">
.map-panel {
    margin: 0 -15px;
}
.map-panel__top {
    .card {
        border-radius: 0;
        border: 0;
        background: #f4f4f4;
        border-bottom: 1px solid #eee;
    }
}
.map-panel__points-list {
    list-style: none;
    margin: 0;
    padding: 0;
}
.map-panel__point {
    border-bottom: 1px solid #eee;
    padding: 10px 20px;
    padding-right: 100px;
    position: relative;
    h3 {
        font-size: 18px;
        margin-bottom: 0;
    }
    small {
        color: #777;
    }
}
.map-panel__point-actions {
    position: absolute;
    top: 0;
    right: 0;
    display: flex;
    align-items: center;
    justify-content: flex-end;
    width: 100px;
    height: 100%;
}
</style>