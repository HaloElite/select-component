<style lang="scss" scoped>
@use "@/styles/components/_inputs";
@use '@/styles/_variables';

// .displayed-value:after {
//     position: absolute;
//     content: "";
//     top: 40%;
//     right: .5rem;
//     width: 0;
//     height: 0;
//     border: 5px solid transparent;
//     border-color: variables.$highlight transparent transparent transparent;
// }

.select-hide {
    display: none;
}

.dropdown-params {
    max-height: 250px;
    -ms-overflow-style: none;
    scrollbar-width: none;
}

.dropdown-params::-webkit-scrollbar {
    display: none;
}

.item:hover {
    background-color: variables.$primary;
    color: variables.$base;
}

.rotate {
    opacity: 1;
    animation: rotateOnAction .25s ease-in forwards;
}

@keyframes rotateOnAction {
    0% {
        transform: rotate(0);
    }

    100% {
        transform: rotate(180deg);
    }
}

.rotateUp {
    opacity: 1;
    animation: rotateUpOnAction .25s ease-in forwards;
}

@keyframes rotateUpOnAction {
    0% {
        transform: rotate(-180deg);
    }

    100% {
        transform: rotate(0);
    }
}
</style>

<template>
<div class="relative text-left outline-none select-none cursor-pointer border border-gray-02 box-border" :tabindex="tabindex" @focusout.stop="onLeaveSelect($event)">
    <div class="flex flex-row justify-start items-center bg-base select-none color-primary main-text">
        <div class="flex-grow" :class="{ 'open': open, 'p-2': !searchActivated }" @click="openStateOnTarget">
            <template v-if="!searchActivated && !multiselect">
                <span class="text-sm color-primary">
                    {{ displayedValue }}
                </span>
            </template>
            <template v-if="!searchActivated && multiselect">
                <template v-if="multiSelectIndexes.length > 0">
                    <div v-for="idx in multiSelectIndexes" :key="idx" class="inline-block bg-highlight text-white text-sm px-2 mx-1 rounded-full">
                        {{ options[idx][labelBy] }}
                    </div>
                </template>
                <template v-else>
                    {{ displayedValue }}
                </template>
            </template>

            <template v-if="searchActivated">
                <input id="searchInputField" type="text" v-model="searchKeyword" @input="searchForOption" placeholder="Suchen..." class="revert-me input-standard-overwrite" style="pointer-event: auto; width: 150px!important; border: none!important;">
            </template>
        </div>

        <div v-if="searchActivated" @click.stop="quitSearch" class="leading-2 bg-error inline-block p-2">
            <img src="@/assets/svg/icon-x-white.svg" class="w-4 h-4 inline-block" alt="delete">
        </div>

        <div v-if="!searchActivated && !noValueSelected && deletable" @click.stop="deleteSelected($event, 'select')" class="leading-2 bg-gray-02 inline-block p-2">
            <img src="@/assets/svg/icon-x.svg" class="w-4 h-4 inline-block" alt="delete">
        </div>

        <div class="bg-highlight inline-block p-2" @click.stop="open = !open">
            <img src="@/assets/svg/icon-arrow-down-base.svg" class="w-4 h-4 inline-block" :class="{'rotate': open, 'rotateUp': !open}" alt="open/close">
        </div>
    </div>

    <div class="absolute left-0 top-full w-full z-30 text-white bg-base cursor-pointer select-none" :class="{ 'select-hide': !open }">
        <div class="flex flex-row">
            <div class="dropdown-params flex-grow overflow-y-scroll overflow-x-hidden bg-base rounded-bl-sm border-l border-r border-b border-gray-01">
                <template v-if="!grouped">
                    <div v-for="(option, index) of options" :key="option.value" @click="openStateOnSelect(false, index); selectEvent(option, 'select', index);">
                        <!-- Show standard dropdown-->
                        <template v-if="!searchActivated">
                            <div class="px-4 py-2 text-sm color-primary item shadow-sm" :class="{'bg-red-200': (lastSelectedIndex === index && !noValueSelected), 'bg-blue-200': multiSelectIndexes.includes(index)}" @mouseenter="descriptionIndex = index; showDescription = true;">
                                {{ option[labelBy] }}
                            </div>
                        </template>

                        <!-- Show search results -->
                        <template v-else-if="searchActivated && searchResults.includes(index)">
                            <div class="px-4 py-2 text-sm color-primary item shadow-sm" :class="{'bg-red-200': (lastSelectedIndex === index && !noValueSelected), 'bg-blue-200': multiSelectIndexes.includes(index)}" @mouseenter="descriptionIndex = index; showDescription = true;">
                                {{ option[labelBy] }}
                            </div>
                        </template>
                    </div>
                </template>

                <template v-if="grouped">
                    <div v-for="(option, index) of options" :key="option.value" @click="openStateOnSelect(false, index); selectEvent(option, 'select', index);">
                        <template v-if="!searchActivated">
                            <!-- Show grouped standard dropdown headers -->
                            <template v-if="option.group">
                                <div @click.capture.stop class="px-4 py-2 font-semibold text-md color-base bg-gray-01 shadow-sm" :class="[option.classNameSpec, option.classNameGen]">
                                    {{ option[labelBy] }}
                                </div>
                            </template>

                            <!-- Show grouped standard dropdown entries -->
                            <template v-else>
                                <div :style="`background-color: ${optionColors[index]}`" class="px-4 py-2 text-sm color-primary item shadow-sm" :class="[option.classNameSpec, option.classNameGen, {'text-red-200': (lastSelectedIndex === index), 'text-blue-300': multiSelectIndexes.includes(index)}]" @mouseenter="descriptionIndex = index; showDescription = true;">
                                    {{ option[labelBy] }}
                                </div>
                            </template>
                        </template>

                        <template v-else-if="searchActivated && searchResults.includes(index)">
                            <!-- Show search results group headers -->
                            <template v-if="option.group">
                                <div @click.capture.stop class="px-4 py-2 font-semibold text-md color-base bg-gray-01 shadow-sm" :class="[option.classNameSpec, option.classNameGen]">
                                    {{ option[labelBy] }}
                                </div>
                            </template>

                            <!-- Show search results group entries -->
                            <template v-else>
                                <div :style="`background-color: ${optionColors[index]}`" class="px-4 py-2 text-sm color-primary item shadow-sm" :class="[option.classNameSpec, option.classNameGen, {'text-red-200': (lastSelectedIndex === index), 'text-blue-300': multiSelectIndexes.includes(index)}]" @mouseenter="descriptionIndex = index; showDescription = true;">
                                    {{ option[labelBy] }}
                                </div>
                            </template>
                        </template>
                    </div>
                </template>
            </div>

            <div v-if="optionDescription && showDescription" class="dropdown-params max-w-250 text-left px-4 py-2 whitespace-normal overflow-y-scroll bg-primary">
                <p>Beschreibung:</p>
                <p class="text-sm text-white">{{ options[descriptionIndex].description }}</p>
            </div>
        </div>
    </div>
</div>
</template>

<script>
import {
    ref,
    reactive,
    watch,
    computed,
    onMounted,
    nextTick
} from 'vue'

export default {
    name: "pk_select",
    props: {
        optionIndexModel: {
            type: Number,
            default: null
        },
        optionIndexModelMultiple: {
            type: Array
        },

        options: {
            type: Array,
        },
        placeholder: {
            type: Object,
            required: false,
            default: null,
        },
        noSelectionText: {
            type: String,
            required: false,
            default: "- Bitte wählen -"
        },
        labelBy: {
            type: String,
            required: true,
            default: "label"
        },
        preselect: {
            type: Boolean,
            required: false,
            default: false
        },
        preselectIndex: {
            type: Array,
            default: [],
            required: false,
        },
        multiselect: {
            type: Boolean,
            required: false,
            default: false
        },
        grouped: {
            type: Boolean,
            required: false,
            default: false
        },
        optionDescription: {
            type: Boolean,
            required: false,
            default: false
        },
        searchable: {
            type: Boolean,
            required: false,
            default: false
        },
        deletable: {
            type: Boolean,
            required: false,
            default: true
        },
        tabindex: {
            type: Number,
            required: false,
            default: 0,
        },
        groupColors: {
            type: Array,
            required: false,
        }
    },
    emits: ['select', ['update:optionIndexModel'],
        ['update:optionIndexModelMultiple']
    ],
    setup(props, {
        emit
    }) {
        // DROPDOWN CONFIG
        var open = ref(false),
            noValueSelected = ref(true),
            searchActivated = ref(false),
            showDescription = ref(false);

        var optionColors = reactive([]);

        const openStateOnSelect = (openState, index) => {
            if (!props.multiselect) {
                open.value = openState;
            }
        }

        const openStateOnTarget = () => {
            if (open.value && props.searchable) {
                showDescription.value = false;
                searchActivated.value = true;
                nextTick(() => {
                    document.getElementById("searchInputField").focus();
                })
            } else {
                open.value = !open.value;
            }
        }

        const onLeaveSelect = (evt) => {
            if (!evt.currentTarget.contains(evt.relatedTarget)) {
                open.value = false;
                showDescription.value = false;
                searchActivated.value = false;
                currentlySearching.value = true;
                searchResults.splice(0);
                searchKeyword.value = "";
            }
        }

        // SELECT-VALUE RELATED
        var selected = ref(null);
        var multiSelected = reactive([]);
        var multiSelectIndexes = reactive([]);

        var descriptionIndex = ref(1);
        var lastSelectedIndex = ref(null);

        var displayedValue = computed(() => {
            if (!props.multiselect && selected.value) {
                return (selected.value[props.labelBy].length > 15 ? selected.value[props.labelBy].substring(0, 15 - 3) + "..." : selected.value[props.labelBy].substring(0, 15));
            } else {
                return props.noSelectionText;
            }
        })

        watch(() => props.optionIndexModel, (newval) => {
            if (newval !== lastSelectedIndex.value && !noValueSelected.value && props.options) {
                selectEvent(props.options[newval], 'select', newval);
            }
        });

        // TODO: Multiselect v-model kopieren, falls meherere Indizes gleichzeitig geändert werden
        // watch(() => props.optionIndexModelMultiple, (newval) => {
        //     if (newval.length) {
        //         multiSelected.splice(0);
        //         multiSelectIndexes.splice(0);

        //         multiSelectIndexes = [...newval];
        //         multiSelectIndexes.forEach((idx) => {
        //             multiSelected[idx] = props.options[idx];
        //         })
        //         console.log(multiSelected);
        //     }
        // });

        // ---------------------------------------------------------------------------------------------------- Select methods
        const selectEvent = (selectedOption, eventName, index) => {
            if (!props.multiselect) {
                if (searchActivated.value) {
                    searchActivated.value = false;
                    searchKeyword.value = "";
                }

                if (lastSelectedIndex.value === index && !noValueSelected.value && props.deletable) {
                    selected.value = null;
                    noValueSelected.value = true;
                    emit('update:optionIndexModel', null);
                } else {
                    selected.value = selectedOption;
                    lastSelectedIndex.value = index;
                    noValueSelected.value = false;
                    emit('update:optionIndexModel', index);
                }

                emit(eventName, selected.value);

            } else if (props.multiselect) {
                if (multiSelectIndexes.includes(index)) {
                    if (!props.deletable && multiSelectIndexes.length == 1) {
                        console.log("test");
                        return;
                    }
                    var position = multiSelectIndexes.indexOf(index);
                    multiSelected.splice(position, 1);
                    multiSelectIndexes.splice(position, 1);

                    if (multiSelectIndexes.length === 0) {
                        noValueSelected.value = true;
                        emit('update:optionIndexModelMultiple', []);
                    }
                } else {
                    multiSelected.push(selectedOption);
                    multiSelectIndexes.push(index);
                    noValueSelected.value = false;
                    emit('update:optionIndexModelMultiple', multiSelectIndexes);
                }

                emit(eventName, multiSelected, index);
            }
        }

        const deleteSelected = (evt, eventName) => {
            noValueSelected.value = true;

            if (!props.multiselect && selected.value) {
                selected.value = "";
                lastSelectedIndex.value = undefined;
                emit('update:optionIndexModel', null);
            } else if (props.multiselect && multiSelectIndexes.length > 0) {
                multiSelected.splice(0);
                multiSelectIndexes.splice(0);
            }

            open.value = false;
            emit(eventName, null);
        }

        // ---------------------------------------------------------------------------------------------------- Searching
        var searchResults = reactive([]);
        var searchKeyword = ref("");
        var currentlySearching = ref(false);

        const searchForOption = (evt) => {
            currentlySearching.value = true;
            searchResults.splice(0);

            var searchString = searchKeyword.value.toLowerCase();

            if (searchString.length > 0) {
                props.options.forEach((element, index) => {
                    if (!element.hasOwnProperty('group')) {
                        if (element.select_label.toLowerCase().indexOf(searchString) > -1) {
                            searchResults.push(index);
                        }
                    }
                });
            }
        }

        const quitSearch = () => {
            currentlySearching.value = true;
            searchActivated.value = false;
            searchResults.splice(0);
            open.value = false;
        }

        onMounted(() => {
            nextTick(() => {
                if (props.preselect && (typeof props.preselectIndex[0] == 'number') && props.preselectIndex[0] !== null && props.options.length) {
                    noValueSelected.value = false;

                    if (props.multiselect) {
                        for (let i = 0; i < props.preselectIndex.length; i++) {
                            if (typeof props.preselectIndex[i] == 'number') {
                                multiSelectIndexes.push(i);
                                multiSelected.push(props.options[i]);
                            }
                        }
                        emit("select", multiSelected);
                    } else if (props.options) {
                        lastSelectedIndex.value = props.preselectIndex[0];
                        selected.value = props.options[props.preselectIndex[0]];
                        emit("select", selected.value);
                    }
                }

                // Dynamically adding group item background colors
                if (props.grouped) {
                    var headers = document.getElementsByClassName(`group-header`);

                    for (var index = 0; index <= headers.length; index++) {
                        var members = document.getElementsByClassName(`group-member-${index+1}`);

                        for (let idx = 0; idx <= [...members].length; idx++) {
                            optionColors.push(props.groupColors[index]);
                        }
                    }
                }
            });
        });

        return {
            displayedValue,
            selected,
            multiSelected,
            optionColors,
            searchResults,
            open,
            searchActivated,
            noValueSelected,
            showDescription,
            descriptionIndex,
            lastSelectedIndex,
            multiSelectIndexes,
            searchKeyword,
            // Methods
            openStateOnSelect,
            openStateOnTarget,
            onLeaveSelect,
            selectEvent,
            deleteSelected,
            searchForOption,
            quitSearch
        }
    }
};
</script>
