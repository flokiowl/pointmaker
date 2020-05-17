<template>
    <div class="map-panel">
        <div class="map-panel__top">
            <markerForm @submit="newPoint" @edit="confirmEdit"/>
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
                    size: 6,
                    weight: 2,
                    color: '#0380e7',
                    position: '',
                    bounds: kievCoord,
                },
                {
                    caption: 'SOFTPRO',
                    type: 'Circle',
                    size: 8,
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
            this.$bvToast.toast('Точку додано', {
                title: 'Успіх!',
                toaster: 'b-toaster-bottom-right',
                variant: 'success',
                solid: true
            })
        },
        setView(point) {
            this.$eventBus.$emit('setView', point)
        },
        saveCsv(point) {
            let data
            let boundResult
            if (point.type === 'Circle') {
                data = [[point.caption], [point.type], [point.size], [point.color], [point.position]]
            } else {
                let newBounds = []
                point.bounds.forEach(b => {
                    newBounds.push(b.toString().replace('[', '').replace(']', ''))
                })
                boundResult = newBounds.join(';')
                data = [[point.caption], [point.type], [point.weight], [point.color], [boundResult]]
            }
            data = data.map(el => {return el[0] + "\r\n"})
            saveAs( new Blob( data, {type : 'data:text/csv;charset=utf-8'}), `${point.caption}.csv`, { autoBom: true } )
        },
        edit(point) {
            this.$eventBus.$emit('edit', point)
        },
        confirmEdit(point, old) {
            console.log('point', point)
            let target = this.points[this.points.indexOf(old)]
            let newBounds = ''
            if (point.bounds !== '') {
                let bounds = point.bounds.trim().split(';')
                newBounds = []
                bounds.forEach(bound => {
                    let boundArr = bound.split(',')
                    newBounds.push([+boundArr[0],+boundArr[1]])
                })
            }
            target.caption = point.caption
            target.type = point.type
            target.position = point.position.split(',')
            target.size = point.size
            target.weight = point.weight
            target.color = point.color
            target.bounds = newBounds
            this.$eventBus.$emit('setPoints', this.points)
            this.$bvToast.toast('Точку оновлено', {
                title: 'Успіх!',
                toaster: 'b-toaster-bottom-right',
                variant: 'success',
                solid: true
            })
        },
        remove(point) {
            this.$bvModal.msgBoxConfirm('Видалити точку?', {okTitle: 'Так', cancelTitle: 'Ні', okVariant: 'danger'})
            .then(value => {
                if (value) {
                    this.points.splice(this.points.indexOf(point), 1)
                    this.$eventBus.$emit('setPoints', this.points)
                    this.$bvToast.toast('Точку видалено', {
                        title: 'Успіх!',
                        toaster: 'b-toaster-bottom-right',
                        variant: 'success',
                        solid: true
                    })
                }
            })
        }
    },
    created() {
        this.$eventBus.$emit('setPoints', this.points)
        this.$eventBus.$emit('editPoint', this.confirmEdit)
    }
}
</script>

<style lang="scss">
.map-panel {
    margin: 0 -15px;
}
.map-panel__top {
    height: 350px;
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
    max-height: calc(100vh - 350px);
    overflow: auto;
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