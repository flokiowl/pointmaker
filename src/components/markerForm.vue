<template>
    <b-card>
        <b-form-group class="mb-0">

            <b-form-group label-cols-sm="2" label="Назва:"  label-align-sm="right" label-for="marker-caption">
                <b-form-input v-model="formData.caption"></b-form-input>
            </b-form-group>

            <b-form-group label-cols-sm="2" label="Тип:" label-align-sm="right" :disabled="editable">
                <b-form-radio-group v-model="formData.type" class="pt-2" :options="['Circle', 'Rectangle', 'Polygon']"></b-form-radio-group>
            </b-form-group>

            <b-form-group label-cols-sm="2" label="Розмір:" label-align-sm="right" v-if="formData.type === 'Circle'">
                <b-form-radio-group v-model="formData.size" class="pt-2" :options="[4, 6, 8]"></b-form-radio-group>
            </b-form-group>

            <b-form-group label-cols-sm="2" label="Товщина:" label-align-sm="right" v-if="formData.type === 'Rectangle' || formData.type === 'Polygon'">
                <b-form-radio-group v-model="formData.weight" class="pt-2" :options="[2, 3, 4]"></b-form-radio-group>
            </b-form-group>

            <b-form-group label-cols-sm="2" label="Колір:" label-align-sm="right">
                <verte model="hex" menuPosition="bottom" v-model="formData.color"></verte>
            </b-form-group>

            <b-form-group label-cols-sm="2" label="Позиція:" label-align-sm="right" v-if="formData.type === 'Circle'">
                <b-form-input v-model="formData.position" placeholder="50.449768, 30.522084"></b-form-input>
            </b-form-group>

            <b-form-group label-cols-sm="2" label="Баунд:" label-align-sm="right" v-if="formData.type === 'Rectangle' || formData.type === 'Polygon'">
                <b-form-input v-model="formData.bounds" placeholder="50.468705, 30.486373; 50.43832, 30.581132"></b-form-input>
            </b-form-group>
            <div class="text-center" v-if="editable">
                <b-button class="mr-2" @click="confirmEdit">Оновити</b-button>
                <b-button type="submit" variant="outline-secondary" @click="cancelEdit">Відмінити</b-button>
            </div>
            <div class="text-center" v-else>
                <b-button type="submit" @click="submit">Додати</b-button>
            </div>

        </b-form-group>
    </b-card>
</template>

<script>
import verte from 'verte'
import 'verte/dist/verte.css'
export default {
    components: {
        verte 
    },
    data() {
        return {
            formData: {
                caption: '',
                type: 'Circle',
                size: 4,
                weight: 2,
                color: '#ec0303',
                position: '',
                bounds: ''
            },
            editable: false,
            oldPoint: null
        }
    },
    methods: {
        submit() {
            this.$emit('submit', this.formData)
            this.setDefault()
        },
        edit(point) {
            this.editable = true
            this.oldPoint = point
            let boundResult = ''
            if (point.bounds !== '') {
                let newBounds = []
                let bounds = point.bounds.forEach(b => {
                    newBounds.push(b.toString().replace('[', '').replace(']', ''))
                })
                boundResult = newBounds.join(';')
            }
            this.formData.caption = point.caption || ''
            this.formData.type = point.type
            this.formData.size = point.size || 4
            this.formData.weight = point.weight || 2
            this.formData.color = point.color
            this.formData.position = point.position.toString().replace('[', '').replace(']', '') || ''
            this.formData.bounds = boundResult
        },
        confirmEdit() {
            this.$emit('edit', this.formData, this.oldPoint)
            this.editable = false
            this.setDefault()
        },
        cancelEdit() {
            this.editable = false
            this.setDefault()
        },
        setDefault() {
            this.formData.caption = ''
            this.formData.type = 'Circle'
            this.formData.size = 4
            this.formData.weight = 2
            this.formData.position = ''
            this.formData.bounds = ''
        }
    },
    mounted() {
        this.$eventBus.$on('edit', this.edit)
    }
}
</script>

<style lang="scss">
    .verte {
        justify-content: flex-start;
        height: 100%;
        align-items: center;
    }
</style>