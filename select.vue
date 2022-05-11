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

.rollup-enter-active {
    animation: roll-in-up .25s;
}

.rollup-leave-active {
    animation: roll-in-up .25s reverse;
}

@keyframes roll-in-up {
    0% {
        transform: scaleY(0);
        opacity: 0;
        transform-origin: bottom;
    }

    100% {
        transform: scaleY(1);
        opacity: 1;
        transform-origin: bottom;
    }
}

.rolldown-enter-active {
    animation: roll-in-down .25s;
}

.rolldown-leave-active {
    animation: roll-in-down .25s reverse;
}

@keyframes roll-in-down {
    0% {
        transform: scaleY(0);
        opacity: 0;
        transform-origin: top;
    }

    100% {
        transform: scaleY(1);
        opacity: 1;
        transform-origin: top;
    }
}

.floating {
    position: absolute;
    top: -0.5rem;
    left: .5rem;
    font-size: .875rem;
    line-height: .875rem;
    color: lighten(variables.$primary, 10%);
    ;
    font-family: variables.$text-2, sans-serif;
    user-select: none;
    background-color: variables.$base;
}
</style>

<template>
<div :id="elementID" class="relative text-left outline-none select-none cursor-pointer border border-gray-02 box-border" @keydown.stop="arrowNavigation($event)">
    <span class="floating">{{ floatingLabel }}</span>
    <div class="flex justify-start items-center bg-base select-none color-primary main-text">
        <div class="flex-grow p-2 w-10/12" :class="{ 'open': open}">
            <template v-if="!multiselect">
                <span class="text-base color-primary">
                    {{ displayedValue }}
                </span>
            </template>
            <template v-if="multiselect">
                <template v-if="multiSelectIndexes.length > 0">
                    <div v-for="(idx, index) in multiSelectIndexes" :key="idx" class="inline-block bg-highlight text-white text-base px-1 mx-1 rounded-sm my-1">
                        {{ options[idx][labelBy] }}
                        <div v-if="multiSelectIndexes.length > 1 || deletable" @click.stop="deleteSelectedElement($event, 'select', index)" class="leading-2 bg-transparent px-1 rounded-full inline-block">
                            <img src="@/assets/svg/icon-x-white.svg" class="w-4 h-4 inline-block" alt="delete">
                        </div>
                    </div>
                </template>
                <template v-else>
                    {{ displayedValue }}
                </template>
            </template>
        </div>

        <div class="flex justify-end flex-grow self-start w-8">
            <div v-if="!noValueSelected && deletable" @click.stop="deleteSelected($event, 'select')" class="leading-2 bg-gray-02 inline-block px-1 py-2">
                <img src="@/assets/svg/icon-x.svg" class="w-4 h-4 inline-block" alt="delete">
            </div>

            <div class="bg-highlight inline-block rounded-bl-sm px-1 py-2">
                <img src="@/assets/svg/icon-arrow-down-base.svg" class="w-4 h-4 inline-block" :class="{'rotate': open, 'rotateUp': !open}" alt="open/close">
            </div>
        </div>
    </div>

    <transition :name="(openingDirection === 'up' ? 'rollup' : 'rolldown')">
        <div v-if="open" class="absolute left-0 w-full z-30 text-white bg-base cursor-pointer select-none" :class="{ 'bottom-full': openingDirection === 'up', 'top-full': openingDirection === 'down' }">

            <div v-if="searchable && openingDirection === 'down'" class="bg-red-100 flex flex-row">
                <input :id="searchUUID" type="text" v-model="searchKeyword" @input="searchForOption" placeholder="Suchen..." class="revert-me input-standard-overwrite" style="pointer-event: auto;">
                <div @click.stop="quitSearch" class="leading-2 bg-error inline-block p-1">
                    <img src="@/assets/svg/icon-x-white.svg" class="w-4 h-4 inline-block" alt="delete">
                </div>
            </div>

            <div class="flex flex-row">
                <div class="dropdown-params flex-grow overflow-y-scroll overflow-x-hidden bg-base rounded-bl-sm border-l border-r border-b border-gray-01">
                    <template v-if="!grouped">
                        <div v-for="(option, index) of options" :key="option.value" @click="openStateOnSelect(false, index); selectEvent(option, 'select', index);">
                            <!-- Show standard dropdown-->
                            <template v-if="!currentlySearching">
                                <div tabindex="-1" class="px-4 py-2 text-base color-primary item shadow-sm outline-none" :class="[`elem-${index}`, {'bg-red-200': index === hasFocus, 'bg-gray-03': (lastSelectedIndex === index && !noValueSelected), 'bg-blue-200': multiSelectIndexes.includes(index)}]" @mouseenter="descriptionIndex = index; showDescription = true;">
                                    {{ option[labelBy] }} {{ hasFocus }}
                                </div>
                            </template>

                            <!-- Show search results -->
                            <template v-else-if="searchResults.includes(index)">
                                <div tabindex="-1" class="px-4 py-2 text-base color-primary item shadow-sm outline-none" :class="[`elem-${index}`, {'bg-red-200': index === searchFocus, 'bg-gray-03': (lastSelectedIndex === index && !noValueSelected), 'bg-blue-200': multiSelectIndexes.includes(index)}]" @mouseenter="descriptionIndex = index; showDescription = true;">
                                    {{ option[labelBy] }} {{ searchFocus }}
                                </div>
                            </template>
                        </div>
                    </template>

                    <template v-if="grouped">
                        <div v-for="(option, index) of options" :key="option.value" @click="openStateOnSelect(false, index); selectEvent(option, 'select', index);">
                            <template v-if="!currentlySearching">
                                <!-- Show grouped standard dropdown headers -->
                                <template v-if="option.group">
                                    <div @click.capture.stop class="px-4 py-2 font-semibold text-md color-base bg-gray-01 shadow-sm" :class="[option.classNameSpec, option.classNameGen]">
                                        {{ option[labelBy] }}
                                    </div>
                                </template>

                                <!-- Show grouped standard dropdown entries -->
                                <template v-else>
                                    <div :style="`background-color: ${optionColors[index]}`" tabindex="-1" class="px-4 py-2 text-base color-primary item shadow-sm outline-none" :class="[option.classNameSpec, option.classNameGen, {'text-red-200': (lastSelectedIndex === index), 'text-blue-300': multiSelectIndexes.includes(index)}]" @mouseenter="descriptionIndex = index; showDescription = true;">
                                        {{ option[labelBy] }}
                                    </div>
                                </template>
                            </template>

                            <template v-else-if="searchResults.includes(index)">
                                <!-- Show search results group headers -->
                                <template v-if="option.group">
                                    <div @click.capture.stop class="px-4 py-2 font-semibold text-md color-base bg-gray-01 shadow-sm" :class="[option.classNameSpec, option.classNameGen]">
                                        {{ option[labelBy] }}
                                    </div>
                                </template>

                                <!-- Show search results group entries -->
                                <template v-else>
                                    <div :style="`background-color: ${optionColors[index]}`" tabindex="-1" class="px-4 py-2 text-base color-primary item shadow-sm outline-none" :class="[option.classNameSpec, option.classNameGen, {'text-red-200': (lastSelectedIndex === index), 'text-blue-300': multiSelectIndexes.includes(index)}]" @mouseenter="descriptionIndex = index; showDescription = true;">
                                        {{ option[labelBy] }}
                                    </div>
                                </template>
                            </template>
                        </div>
                    </template>
                </div>

                <div v-if="optionDescription && showDescription" class="dropdown-params max-w-250 text-left px-4 py-2 whitespace-normal overflow-y-scroll bg-primary">
                    <p>Beschreibung:</p>
                    <p class="text-base text-white">{{ options[descriptionIndex].description }}</p>
                </div>
            </div>

            <div v-if="searchable && openingDirection === 'up'" class="bg-red-100 flex flex-row">
                <input :id="searchUUID" type="text" v-model="searchKeyword" @input="searchForOption" placeholder="Suchen..." class="revert-me input-standard-overwrite" style="pointer-event: auto;">
                <div @click.stop="quitSearch" class="leading-2 bg-error inline-block p-1">
                    <img src="@/assets/svg/icon-x-white.svg" class="w-4 h-4 inline-block" alt="delete">
                </div>
            </div>
        </div>
    </transition>
</div>
</template>

<script>
import {
    ref,
    reactive,
    watch,
    computed,
    onMounted,
    onBeforeUnmount,
    nextTick
} from 'vue'

import {
    v4 as uuidv4
} from 'uuid';

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
            required: true
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
        openingDirection: {
            type: String,
            required: false,
            default: "down"
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
        groupColors: {
            type: Array,
            required: false,
        },
        floatingLabel: {
            type: String,
            required: false
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
            showDescription = ref(false),
            deletable = ref(false),
            searchUUID = uuidv4(),
            elementID = uuidv4();

        var optionColors = reactive([]);

        const openStateOnSelect = (openState, index) => {
            if (!props.multiselect) {
                open.value = openState;
            }
        }

        const openCloseSelect = (evt) => {
            if (open.value) {
                open.value = false;
                showDescription.value = false;
                currentlySearching.value = false;
                searchResults.splice(0);
                searchKeyword.value = "";
            } else {
                open.value = true;
                nextTick(() => {
                    // Focus search input field
                    setTimeout(() => {
                        if (lastSelectedIndex.value) {
                            document.getElementsByClassName(`elem-${lastSelectedIndex.value}`)[0].focus();
                        } else {
                            document.getElementsByClassName(`elem-0`)[0].focus();
                            if (currentlySearching.value) {
                                searchFocus.value = 0;
                            } else {
                                hasFocus.value = 0;
                            }

                        }
                    }, 100);
                });
            }
        }

        // SELECT-VALUE RELATED
        var selected = computed(() => {
            if (!noValueSelected.value && typeof lastSelectedIndex.value === 'number') {
                return props.options[lastSelectedIndex.value];
            } else {
                return "";
            }
        });

        var multiSelected = computed(() => {
            if (!noValueSelected.value) {
                var temp = [];
                multiSelectIndexes.value.forEach((el, idx) => {
                    temp.push(props.options[index]);
                });

                return temp;
            } else {
                return [];
            }
        });

        var multiSelectIndexes = reactive([]);

        var descriptionIndex = ref(1);
        var lastSelectedIndex = ref(null);
        var hasFocus = ref(null),
            searchFocus = ref(0);

        var displayedValue = computed(() => {
            if (!props.multiselect && selected.value) {
                return (selected.value[props.labelBy].length > 15 ? selected.value[props.labelBy].substring(0, 15 - 3) + "..." : selected.value[props.labelBy].substring(0, 15));
            } else {
                return props.noSelectionText;
            }
        })

        watch(() => props.optionIndexModel, (newval) => {
            if (typeof newval === 'number' && newval !== lastSelectedIndex.value && !noValueSelected.value && props.options) {
                // selectEvent(props.options[newval], 'select', newval);
                lastSelectedIndex.value = newval;
                hasFocus.value = newval;
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

                if (lastSelectedIndex.value === index && !noValueSelected.value && deletable.value) {
                    noValueSelected.value = true;
                    emit('update:optionIndexModel', null);
                } else if (lastSelectedIndex.value === index && !noValueSelected.value && !deletable.value) {
                    return;
                } else {
                    lastSelectedIndex.value = index;
                    hasFocus.value = index;
                    noValueSelected.value = false;
                    emit('update:optionIndexModel', index);
                }

                emit(eventName, selected.value);

            } else if (props.multiselect) {
                if (multiSelectIndexes.includes(index)) {
                    if (!deletable.value && multiSelectIndexes.length == 1) {
                        return;
                    }
                    var position = multiSelectIndexes.indexOf(index);
                    multiSelectIndexes.splice(position, 1);

                    if (multiSelectIndexes.length === 0) {
                        noValueSelected.value = true;
                        emit('update:optionIndexModelMultiple', []);
                    }
                } else {
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
                lastSelectedIndex.value = undefined;
                hasFocus.value = 0;
                emit('update:optionIndexModel', null);
            } else if (props.multiselect && multiSelectIndexes.length > 0) {
                multiSelectIndexes.splice(0);
            }

            open.value = false;
            emit(eventName, null);
        }

        const deleteSelectedElement = (evt, eventName, index) => {
            if (!deletable.value && multiSelectIndexes.length == 1) {
                return;
            } else if (props.multiselect && multiSelectIndexes.length > 0) {
                multiSelectIndexes.splice(index, 1);
                emit(eventName, multiSelected, index);
            }
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
                searchFocus.value = searchResults[0];
            } else {
                currentlySearching.value = false;
                searchFocus.value = 0;
            }
        }

        const quitSearch = () => {
            currentlySearching.value = false;
            searchResults.splice(0);
        }

        // ---------------------------------------------------------------------------------------------------- Arrow navigation
        const arrowNavigation = (evt) => {

            // ArrowUp === up  
            // ArrowDown === down
            // Enter === enter
            if (currentlySearching.value && searchResults.length > 0) {
                if (evt.code === "ArrowUp" && searchResults.indexOf(searchFocus.value) > 0) {
                    searchFocus.value = searchResults[(searchResults.indexOf(searchFocus.value) - 1)];
                } else if (evt.code === "ArrowDown" && searchResults.indexOf(searchFocus.value) < (searchResults.length - 1)) {
                    searchFocus.value = searchResults[(searchResults.indexOf(searchFocus.value) + 1)];
                } else if (evt.code === "Enter") {
                    selectEvent(props.options[searchFocus.value], 'select', searchFocus.value);
                    open.value = false;
                }

                if (evt.code !== "ArrowUp" && evt.code !== "ArrowDown" && evt.code !== "Enter") {
                    document.getElementById(searchUUID).focus();
                } else {
                    try {
                        document.getElementsByClassName(`elem-${searchFocus.value}`)[0].focus();
                    } catch (error) {
                        return;
                    }
                }
            } else {
                if (evt.keyCode === 38 && hasFocus.value > 0) {
                    hasFocus.value -= 1;
                } else if (evt.keyCode === 40 && hasFocus.value < (props.options.length - 1)) {
                    hasFocus.value += 1;
                } else if (evt.keyCode === 13) {
                    selectEvent(props.options[hasFocus.value], 'select', hasFocus.value);
                    open.value = false;
                }
                if (evt.code !== "ArrowUp" && evt.code !== "ArrowDown" && evt.code !== "Enter") {
                    document.getElementById(searchUUID).focus();
                } else {
                    try {
                        document.getElementsByClassName(`elem-${hasFocus.value}`)[0].focus();
                    } catch (error) {
                        return;
                    }
                }
            }

        }

        onMounted(() => {
            nextTick(() => {
                if (props.preselect && (typeof props.preselectIndex[0] == 'number') && props.preselectIndex[0] !== null && props.options.length) {
                    noValueSelected.value = false;

                    if (props.multiselect) {
                        for (let i = 0; i < props.preselectIndex.length; i++) {
                            if (typeof props.preselectIndex[i] == 'number') {
                                multiSelectIndexes.push(i);
                            }
                        }
                        emit("select", multiSelected);
                    } else if (props.options) {
                        lastSelectedIndex.value = props.preselectIndex[0];
                        hasFocus.value = props.preselectIndex[0];
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

            document.addEventListener('click', (evt) => {
                var root = document.getElementById(elementID);

                if (root.contains(evt.target)) {
                    openCloseSelect(evt);
                } else {
                    open.value = false;
                }
            });
        });

        onBeforeUnmount(() => {
            document.removeEventListener('click', () => {
                open.value = false
            });
        });

        return {
            displayedValue,
            selected,
            multiSelected,
            optionColors,
            searchResults,
            searchUUID,
            elementID,
            currentlySearching,
            open,
            deletable,
            noValueSelected,
            showDescription,
            descriptionIndex,
            lastSelectedIndex,
            hasFocus,
            searchFocus,
            multiSelectIndexes,
            searchKeyword,
            // Methods
            openStateOnSelect,
            openCloseSelect,
            selectEvent,
            deleteSelected,
            deleteSelectedElement,
            searchForOption,
            quitSearch,
            arrowNavigation
        }
    }
};
</script>
