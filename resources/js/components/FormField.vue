<template>
    <default-field
        :errors="errors"
        :field="field"
        :full-width-content="true"
        :show-errors="false"
    >
        <template slot="field">
            <draggable
                @end="drag=false"
                @start="drag=true"
                handle=".relationship-item-handle"
                v-model="items"
            >
                <relationship-form-item
                    :errors="errorList"
                    :field="field"
                    :id="index"
                    :key="items.id"
                    :model-id="field.models[index]||0"
                    :model-key="field.modelKey"
                    :ref="index"
                    :value="items.fields"
                    @deleted="removeItem(index)"
                    v-for="(items, index) in items"
                />
            </draggable>
            <div
                class="bg-30 flex p-2 border-b border-40 rounded-lg"
                v-if="!field.singular || !items.length"
            >
                <div class="w-full text-right">
                    <span v-if="field.max">{{items.length}} {{__('of')}} {{field.max}}</span>
                    <button
                        :disabled="disabledBtn"
                        @click="addItem()"
                        class="btn btn-default bg-transparent hover:bg-primary text-primary hover:text-white border border-primary hover:border-transparent inline-flex items-center relative mr-3"
                        type="button"
                    >
                        {{__('Add')}} {{ field.singularLabel.toLowerCase() }}
                    </button>
                </div>
            </div>
        </template>
    </default-field>
</template>

<script>
	import {Errors, FormField, HandlesValidationErrors} from 'laravel-nova'
	import draggable from 'vuedraggable'
	import RelationshipFormItem from './RelationshipFormItem.vue'
	
	export default {
		components: {
			draggable,
			RelationshipFormItem
		},
		
		mixins: [FormField, HandlesValidationErrors],
		
		props: ['resourceName', 'resourceId', 'field'],
		
		data: function () {
			return {
				id: 0,
				items: [],
				errorList: new Errors()
			}
		},
		
		watch: {
			errors: function (errors) {
				let errObj = errors.errors.hasOwnProperty(this.field.attribute) ? errors.errors[this.field.attribute][0] : {};
				Object.keys(errObj).forEach(key => {
					errObj[key.replace(/\./g, '_')] = errObj[key];
					delete errObj[key];
				});
				this.errorList = new Errors(errObj);
			},
		},
		
		computed: {
			valueAsArray: function () {
				return Array.isArray(this.items) ? this.items : [];
			},
			disabledBtn: function () {
				return this.field.max ? this.items.length >= this.field.max : false;
			}
		},
		
		methods: {
			/*
             * Set the initial, internal value for the field.
             */
			setInitialValue() {
				this.items = Array.isArray(this.field.value) ? this.field.value : [];
				this.items = this.items.map(item => {
					return {'id': this.getNextId(), 'fields': item}
				});
				
				if (this.field.singular) {
					this.items.splice(1);
				}
				
				if (this.field.addChildAtStart && (this.items.length === 0)) {
					this.items.push({'id': this.getNextId(), 'fields': {...this.field.settings}});
				}
			},
			
			/**
			 * Fill the given FormData object with the field's internal value.
			 */
			fill(formData) {
				try {
					this.fillValueFromChildren(formData)
				} catch (error) {
					console.log(error);
				}
			},
			
			fillValueFromChildren: function (formData) {
				if (!_.isEmpty(this.$refs[0])) {
					_(this.$refs).each(item => {
						if (item && Array.isArray(item) && item[0]) {
							item[0].fill(formData, this.field.attribute);
						}
					});
				} else {
					formData.append(this.field.attribute, []);
				}
			},
			
			/**
			 * Update the field's internal value.
			 */
			handleChange(value) {
				this.items = Array.isArray(value) ? value : [];
			},
			
			getNextId() {
				this.id += 1;
				return this.id;
			},
			
			removeItem(index) {
				let value = [...this.items];
				value.splice(index, 1);
				this.handleChange(value);
			},
			
			addItem() {
				let value = [...this.items];
				value.push({'id': this.getNextId(), 'fields': {...this.field.settings}});
				this.handleChange(value);
			},
		}
	}
</script>
