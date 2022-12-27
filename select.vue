<template>
<div>
    <label :for="id" v-if="grouped">
        <input style="display: none;" :type="type" :id="id" :name="name" :value="valuestorage">
        <PkSelect :deactivated="readonly" :showBorder="hasFocus" :floatingLabel="label" :noSelectionText="noSelectionText" :deletable="deletable" :hasError="errorMessage.length > 0" :grouped="true" v-model:optionIndexModel="modelIndex" :options="selectOptions" :groupColors="groupColors" :preselect="preSelect" :preselectIndex="preselectIndices" :multiselect="false" :searchable="true" :optionDescription="description" :label-by="labelBy" @select="selectHandler" @isFocused="$emit('focused', true)"></PkSelect>
    </label>
{{ modelIndex }}
    <label :for="id" v-if="!grouped">
        <input style="display: none;" :type="type" :id="id" :name="name" :value="valuestorage" :readonly="readonly">
        <PkSelect :deactivated="readonly" :showBorder="hasFocus" :floatingLabel="label" :noSelectionText="noSelectionText" :deletable="deletable" :hasError="errorMessage.length > 0" :grouped="false" v-model:optionIndexModel="modelIndex" :options="selectOptions" :preselect="preSelect" :preselectIndex="preselectIndices" :multiselect="false" :searchable="true" :label-by="labelBy" @select="selectHandler" @isFocused="$emit('focused', true)"></PkSelect>
    </label>

    <template v-for="error in itemErrors" :key="error[errorAttribute]">
        <p v-if="error[errorAttribute] && !valuestorage" class="negative sub-text tw-text-sm tw-py-1">{{ error[errorAttribute] }}</p>
    </template>
</div>
</template>

<script>
import {
    useStore
} from 'vuex'
import {
    onMounted,
    computed,
    watch,
    ref
} from "vue";

import PkSelect from "../../pk_elements/pk_inputs/PkSelect.vue";

export default {
    name: "pk_select",
    props: {
        name: {
            type: String,
            default: ""
        },
        placeholder: {
            type: String,
            default: ""
        },
        refresh: {
            type: Boolean,
            default: false
        },
        preselectIndices: {
            type: Array
        },
        grouped: {
            type: Boolean,
            default: false
        },
        preSelect: {
            type: Boolean,
            default: false
        },
        noSelectionText: {
            type: String,
            required: false,
            default: "- Bitte wählen -"
        },
        label: {
            type: String,
            default: ""
        },
        labelBy: {
            type: String,
            default: ""
        },
        id: {
            type: String,
            default: ""
        },
        type: {
            type: String,
            default: "text"
        },
        errorAttribute: {
            type: String,
            default: ""
        },
        description: {
            type: Boolean,
            default: false
        },
        deletable: {
            type: Boolean,
            default: false
        },
        groupColors: {
            type: Array,
            required: false,
        },
        hasFocus: {
            type: Boolean,
            default: true
        },
        readonly: {
            type: Boolean,
            default: false
        },
        index: {
            type: Number,
            default: 0
        },
        uuid: {
            type: String,
            default: ""
        },
        itemData: JSON,
        selectOptions: Array,
        itemErrors: Array
    },
    emits: ['inputRequest', 'focused'],

    components: {
        PkSelect
    },
    setup(props, {
        emit
    }) {
        const store = useStore();

        var valuestorage = ref();
        var toggled = ref(false);
        var modelIndex = ref(null);

        var errorMessage = computed(() => {
            var temp = "";

            props.itemErrors.forEach(error => {
                if (error[props.errorAttribute]) {
                    temp = error[props.errorAttribute];
                }
            });

            return temp;
        });

        var updateState = computed(() => {
            return store.getters['expense/getStates'].suggestUpdated;
        });

        watch(() => props.itemData, (newval) => {
            valuestorage.value = newval.value;

            if (props.grouped) {
                var idx = -1;
                if (!newval.value) {
                    modelIndex.value = null;
                } else {
                    for (const entry of props.selectOptions) {
                        if (!entry.hasOwnProperty("group")) {
                            idx++;
                            if (entry.select_value === newval.value) {
                                modelIndex.value = idx;
                                break;
                            }
                        }
                    }
                }
            } else {
                if (!newval.value || newval.value.length === 0) {
                    for (const [index, entry] of props.selectOptions.entries()) {
                        if (entry.select_value.toString() === newval.default_value.toString()) {
                            modelIndex.value = index;
                            valuestorage.value = entry.select_value;
                            break;
                        }
                    }
                } else {
                    for (const [index, entry] of props.selectOptions.entries()) {
                        if (entry.select_value.toString() === newval.value.toString()) {
                            modelIndex.value = index;
                            valuestorage.value = entry.select_value;
                        }
                    }
                }
            }

            if (!updateState.value) {
                store.commit('expense/SET_SUGGEST_UPDATED', {
                    val: true,
                    attr: props.itemData.attribute
                });
            }
        }, {
            deep: true
        });

        const selectHandler = (data) => {
            if (data && data.hasOwnProperty("select_value") && props.itemData.attribute !== "depreciation_afa_id") {
                valuestorage.value = data.select_value;
            } else if (props.itemData.attribute === "depreciation_afa_id") {
                valuestorage.value = data.select_value;
                store.commit('expense/MANIPULATE_DEPRECATION_L_Y', {
                    value: data.life_years,
                    uuid: props.uuid,
                    index: props.index
                });
            } else {
                if (data) {
                    valuestorage.value = data.value;
                } else {
                    valuestorage.value = null;
                }
            }

            emit('inputRequest', props.refresh)
        }

        onMounted(() => {
            if (props.preSelect) {
                try {
                    if (!props.grouped) {
                        valuestorage.value = props.selectOptions[0].select_value;
                    } else if (props.grouped) {
                        if (props.selectOptions.length) {
                            let filtered = props.selectOptions.filter((el) => {
                                if (!el.hasOwnProperty("group")) {
                                    return el;
                                }
                            });

                            valuestorage.value = filtered[0].select_value;
                        }
                    }
                    emit('inputRequest', props.refresh);
                } catch (error) {
                    console.log(error);
                }
            }
        })

        return {
            valuestorage,
            errorMessage,
            toggled,
            modelIndex,
            updateState,
            // Methods
            selectHandler
        }
    }
};
</script>
