<style lang="scss" scoped>
@use "@/styles/components/_inputs";
@use '@/styles/_variables';

.input-select-search {
    min-width: 100px;
    width: 100% !important;
    height: 2rem !important;
    box-sizing: border-box !important;
    margin: 0 !important;
    border: none !important;
    padding: 0rem 0.5rem !important;
    line-height: 1.25rem !important;
    font-family: variables.$text-2, "sans-serif";
    font-size: .875rem !important;
    color: variables.$color-primary !important;
    cursor: text;
    outline: none !important;
    background-color: white !important;
}

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
    background-color: variables.$highlight;
    color: variables.$base;
}

.hover-color {
    background-color: variables.$highlight;
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
    left: 0.5rem;
    font-size: .8rem;
    line-height: 0.8rem;
    color: lighten(variables.$primary, 10%);
    font-family: variables.$text-2, sans-serif;
    user-select: none;
    background-color: variables.$base;
}

.border-error {
    border-color: transparent;
    border: 1px variables.$negative solid;
}
</style>

<template>
<div :id="elementID" class="relative w-full outline-none select-none cursor-pointer box-border rounded-md" tabindex="0" @keydown.stop="arrowNavigation($event)">
    <span class="floating">{{ floatingLabel }}</span>
    <div class="flex justify-start bg-base select-none color-primary sub-text border border-gray-02 rounded-md" :class="{'border-error': hasError, 'border-highlight': open}" @click="openCloseSelect($event)">
        <div class="flex-grow w-10/12 rounded-l-sm p-1" :class="{ 'open': open}">
            <template v-if="!multiselect">
                <div class="p-2">
                    <span class="text-sm color-primary" :class="{'color-gray-01': noValueSelected}">
                        {{ displayedValue }}
                    </span>
                </div>
            </template>
            <template v-if="multiselect">
                <div v-if="multiSelectIndexes.length > 0">
                    <template v-for="(idx, index) in multiSelectIndexes" :key="idx">
                        <div v-if="index < 2" class="inline-block bg-main-dark color-primary text-sm p-2 mr-1 rounded-md">
                            {{ options[idx][labelBy].length > 10 ? options[idx][labelBy].substring(0, 10 - 3) + "..." : options[idx][labelBy] }}
                            <div v-if="(multiSelectIndexes.length > 1 || deletable)" @click.stop="deleteSelectedElement($event, 'select', index)" class="leading-2 bg-transparent px-1 rounded-full inline-block">
                                <img src="@/assets/svg/icon-x-gray.svg" class="w-4 h-4 inline-block" alt="delete">
                            </div>
                        </div>
                        <div v-else-if="index === (multiSelectIndexes.length - 1)" class="inline-block color-primary text-sm p-2">
                            + {{ (index - 1) }}
                        </div>
                    </template>
                </div>
                <div v-else>
                    <div class="p-2">
                        <span class="text-sm color-primary" :class="{'color-gray-01': noValueSelected}">
                            {{ displayedValue }}
                        </span>
                    </div>
                </div>
            </template>
        </div>

        <div class="flex justify-end flex-grow bg-base w-8 rounded-r-md">
            <div v-if="!noValueSelected && deletable" @click.stop="deleteSelected($event, 'select')" class="px-1 py-1 bg-white flex items-center">
                <img src="@/assets/svg/icon-x-gray.svg" class="w-4 h-4 inline-block" alt="delete">
            </div>

            <div class="bg-white flex items-center px-1 py-1 rounded-r-md">
                <img src="@/assets/svg/icon-arrow-rounded-down-gray.svg" class="w-3 h-3 inline-block" :class="{'rotate': open, 'rotateUp': !open}" alt="open/close">
            </div>
        </div>
    </div>

    <transition :name="(openingDirection === 'up' ? 'rollup' : 'rolldown')">
        <div v-if="open" class="absolute left-0 w-full z-30 text-white bg-base cursor-pointer border border-gray-02 rounded-md" :class="{ 'bottom-full': openingDirection === 'up', 'top-full': openingDirection === 'down' }">

            <div v-if="searchable && openingDirection === 'down'" class="px-4 my-2">
                <div class="flex flex-row border-b-2 border-gray-02">
                    <div class="bg-white flex items-center px-1 py-1 rounded-r-md">
                        <img src="@/assets/svg/icon-lens.svg" class="w-3 h-3 inline-block" alt="search">
                    </div>
                    <div class="flex-grow">
                        <input :id="searchUUID" type="text" v-model="searchKeyword" @input="searchForOption" placeholder="Liste filtern" class="revert-me input-select-search">
                    </div>
                    <div @click.stop="quitSearch" class="flex justify-center items-center">
                        <img src="@/assets/svg/icon-x-gray.svg" class="w-4 h-4" alt="delete">
                    </div>
                </div>
            </div>

            <div class="flex flex-row">
                <div id="scrollBox" class="dropdown-params flex-grow overflow-y-scroll overflow-x-hidden bg-base" :class="{'rounded-t-md': openingDirection === 'up', 'rounded-b-md': openingDirection === 'down'}">
                    <template v-if="!grouped">
                        <div v-for="(option, index) of options" :key="option.value" @click="selectEvent(option, 'select', index);">
                            <!-- Show standard dropdown-->
                            <template v-if="!currentlySearching">
                                <div @mouseover="hasFocus = index" class="px-4 py-2 text-sm color-primary item shadow-sm outline-none" :class="[`elem-${index}`, {'hover-color': index === hasFocus, 'bg-gray-03': (lastSelectedIndex === index && !noValueSelected), 'bg-main-dark': multiSelectIndexes.includes(index)}]" @mouseenter="descriptionIndex = index; showDescription = true;">
                                    {{ option[labelBy] }}
                                </div>
                            </template>

                            <!-- Show search results -->
                            <template v-else-if="searchResults.includes(index)">
                                <div @mouseover="searchFocus = index" class="px-4 py-2 text-sm color-primary item shadow-sm outline-none" :class="[`elem-${index}`, {'hover-color': index === searchFocus, 'bg-gray-03': (lastSelectedIndex === index && !noValueSelected), 'bg-main-dark': multiSelectIndexes.includes(index)}]" @mouseenter="descriptionIndex = index; showDescription = true;">
                                    {{ option[labelBy] }}
                                </div>
                            </template>
                        </div>
                    </template>

                    <template v-if="grouped">
                        <div v-for="(option, index) of options" :key="option.value" @click="selectEvent(option, 'select', index);">
                            <template v-if="!currentlySearching">
                                <!-- Show grouped standard dropdown headers -->
                                <template v-if="option.group">
                                    <div @click.capture.stop class="px-4 py-2 font-semibold text-md bg-main-dark color-highlight shadow-sm" :class="[`elem-${index}`, {'hover-color': index === hasFocus}, option.classNameSpec, option.classNameGen]">
                                        {{ option[labelBy] }}
                                    </div>
                                </template>

                                <!-- Show grouped standard dropdown entries -->
                                <template v-else>
                                    <div :style="`background-color: ${optionColors[index]}`" class="px-4 py-2 text-sm color-primary item shadow-sm outline-none" :class="[`elem-${index}`, {'hover-color': index === hasFocus}, option.classNameSpec, option.classNameGen, {'text-red-200': (lastSelectedIndex === index), 'text-blue-300': multiSelectIndexes.includes(index)}]" @mouseenter="descriptionIndex = index; showDescription = true;">
                                        {{ option[labelBy] }}
                                    </div>
                                </template>
                            </template>

                            <template v-else-if="searchResults.includes(index)">
                                <!-- Show search results group headers -->
                                <template v-if="option.group">
                                    <div @click.capture.stop class="px-4 py-2 font-semibold text-md bg-main-dark color-highlight shadow-sm" :class="[`elem-${index}`, {'hover-color': index === searchFocus}, option.classNameSpec, option.classNameGen]">
                                        GROUP: {{ option[labelBy] }}
                                    </div>
                                </template>

                                <!-- Show search results group entries -->
                                <template v-else>
                                    <div :style="`background-color: ${optionColors[index]}`" class="px-4 py-2 text-sm color-primary item shadow-sm outline-none" :class="[`elem-${index}`, {'hover-color': index === searchFocus}, option.classNameSpec, option.classNameGen, {'text-red-200': (lastSelectedIndex === index), 'text-blue-300': multiSelectIndexes.includes(index)}]" @mouseenter="descriptionIndex = index; showDescription = true;">
                                        ITEM: {{ option[labelBy] }}
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

            <div v-if="searchable && openingDirection === 'up'" class="px-4 my-2">
                <div class="flex flex-row">
                    <div class="bg-white flex items-center px-1 py-1 rounded-r-md">
                        <img src="@/assets/svg/icon-lens.svg" class="w-3 h-3 inline-block" alt="search">
                    </div>
                    <div class="flex-grow">
                        <input :id="searchUUID" type="text" v-model="searchKeyword" @input="searchForOption" placeholder="Liste filtern" class="revert-me input-select-search" style="pointer-event: auto;">
                    </div>
                    <div @click.stop="quitSearch" class="flex justify-center items-center">
                        <img src="@/assets/svg/icon-x-gray.svg" class="w-4 h-4" alt="delete">
                    </div>
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
        deletable: {
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
        },
        hasError: {
            type: Boolean,
            required: false,
            default: false
        },
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
            searchUUID = uuidv4(),
            elementID = uuidv4();

        var optionColors = reactive([]);

        const closedState = () => {
            open.value = false;
            showDescription.value = false;
            currentlySearching.value = false;
            searchResults.splice(0);
            searchKeyword.value = "";
        }

        const openCloseSelect = (evt, pos) => {
            if (open.value && !props.multiselect) {
                closedState();
            } else {
                open.value = true;
                nextTick(() => {
                    setTimeout(() => {
                        // If a select was made
                        if (lastSelectedIndex.value) {
                            hasFocus.value = lastSelectedIndex.value;
                            document.getElementsByClassName(`elem-${lastSelectedIndex.value}`)[0].focus();
                            scrollToFocusedElement();
                        } else {
                            // If no element was selected
                            document.getElementsByClassName(`elem-0`)[0].focus();
                            if (currentlySearching.value) {
                                searchFocus.value = 0;
                            } else {
                                hasFocus.value = 0;
                            }

                        }
                    }, 150);
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
            if (!props.multiselect && selected.value && selected.value[props.labelBy].length) {
                return (selected.value[props.labelBy].length > 15 ? selected.value[props.labelBy].substring(0, 20 - 3) + "..." : selected.value[props.labelBy].substring(0, 20));
            } else {
                return props.noSelectionText;
            }
        })

        watch(() => props.optionIndexModel, (newval) => {
            // TODO: If grouped -> deliver the correct index for newval as prop for optionIndexModel!
            if (typeof newval === 'number' && newval !== lastSelectedIndex.value && !noValueSelected.value && props.options) {
                lastSelectedIndex.value = newval;
                hasFocus.value = newval;
            }
        });

        // TODO: Multiselect v-model kopieren, falls mehrere Indizes in parent-component gleichzeitig geändert werden
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
                if (lastSelectedIndex.value === index && !noValueSelected.value && props.deletable) {
                    noValueSelected.value = true;
                    hasFocus.value = 0;
                    searchKeyword.value = "";
                    emit('update:optionIndexModel', null);
                } else if (lastSelectedIndex.value === index && !noValueSelected.value && !props.deletable) {
                    return;
                } else {
                    lastSelectedIndex.value = index;
                    hasFocus.value = index;
                    noValueSelected.value = false;
                    emit('update:optionIndexModel', index);
                }

                try {
                    emit(eventName, selected.value);
                    open.value = false;
                } catch (error) {
                    console.log("Something went wrong:", error);
                    return;
                }
            } else if (props.multiselect) {
                if (multiSelectIndexes.includes(index)) {
                    if (!props.deletable && multiSelectIndexes.length == 1) {
                        return;
                    }
                    var position = multiSelectIndexes.indexOf(index);
                    multiSelectIndexes.splice(position, 1);

                    if (multiSelectIndexes.length === 0) {
                        noValueSelected.value = true;
                        hasFocus.value = 0;
                        searchKeyword.value = "";
                        emit('update:optionIndexModelMultiple', []);
                    }
                } else {
                    multiSelectIndexes.push(index);
                    noValueSelected.value = false;
                    emit('update:optionIndexModelMultiple', multiSelectIndexes);
                }

                try {
                    emit(eventName, multiSelected, index);
                } catch (error) {
                    console.log("SELECTED:", selected.value[props.labelBy].substring(0, 15 - 3));
                    console.log("Something went wrong:", error);
                    return;
                }
            }
        }

        const deleteSelected = (evt, eventName) => {
            noValueSelected.value = true;

            if (!props.multiselect && typeof lastSelectedIndex.value === 'number') {
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
            if (!props.deletable && multiSelectIndexes.length == 1) {
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
                if (!props.grouped) {
                    props.options.forEach((element, index) => {
                        if (!element.hasOwnProperty('group')) {
                            if (element.select_label.toLowerCase().indexOf(searchString) > -1) {
                                searchResults.push(index);
                            }
                        }
                    });
                    searchFocus.value = searchResults[0];
                } else if (props.grouped) {
                    props.options.forEach((item, index) => {
                        if (!item?.group && item.select_label.toLowerCase().indexOf(searchString) > -1) {
                            searchResults.push(index);
                        } else if (item?.group) {
                            searchResults.push(index);
                        }
                    });
                }
            } else {
                currentlySearching.value = false;
                searchFocus.value = 0;
            }
        }

        const quitSearch = () => {
            currentlySearching.value = false;
            searchKeyword.value = "";
            searchResults.splice(0);
        }

        // ---------------------------------------------------------------------------------------------------- Arrow navigation
        const scrollToFocusedElement = () => {
            var scrollBox = document.getElementById("scrollBox");
            var x = scrollBox.getBoundingClientRect().x;
            var y = (document.getElementsByClassName(`elem-${hasFocus.value}`)[0].offsetHeight * hasFocus.value) - 2 * (document.getElementsByClassName(`elem-${hasFocus.value}`)[0].offsetHeight);
            scrollBox.scrollTo(x, y);
        }

        const arrowNavigation = (evt) => {
            if (evt.code === "Escape") {
                closedState();
            }

            var scrollBox = document.getElementById("scrollBox");

            if (currentlySearching.value && searchResults.length > 0) {
                if (evt.code === "ArrowUp" && searchResults.indexOf(searchFocus.value) > 0) {
                    searchFocus.value = searchResults[(searchResults.indexOf(searchFocus.value) - 1)];

                    if (props.options[searchFocus.value]?.group && searchFocus.value > 0) {
                        searchFocus.value = searchResults[(searchResults.indexOf(searchFocus.value) - 1)];
                    }
                } else if (evt.code === "ArrowDown" && searchResults.indexOf(searchFocus.value) < (searchResults.length - 1)) {
                    searchFocus.value = searchResults[(searchResults.indexOf(searchFocus.value) + 1)];

                    if (props.options[searchFocus.value]?.group && searchFocus.value < searchResults.length) {
                        searchFocus.value = searchResults[(searchResults.indexOf(searchFocus.value) + 1)];
                    }
                } else if (evt.code === "Enter" && open.value) {
                    selectEvent(props.options[searchFocus.value], 'select', searchFocus.value);
                    if (!props.multiselect) {
                        closedState();
                    }
                }

                if (evt.code !== "ArrowUp" && evt.code !== "ArrowDown" && evt.code !== "Enter" && open.value) {
                    document.getElementById(searchUUID).focus();
                } else if (open.value) {
                    evt.preventDefault();
                    try {
                        var x = scrollBox.getBoundingClientRect().x;
                        var y = (document.getElementsByClassName(`elem-${searchFocus.value}`)[0].offsetHeight * searchResults.indexOf(searchFocus.value)) - 2 * (document.getElementsByClassName(`elem-${searchFocus.value}`)[0].offsetHeight);
                        scrollBox.scrollTo(x, y);
                    } catch (error) {
                        console.log(error);
                        return;
                    }
                } else {
                    return;
                }
            } else {
                if (evt.code === "ArrowUp" && hasFocus.value > 0) {
                    hasFocus.value -= 1;
                    if (props.options[hasFocus.value]?.group && hasFocus.value > 0) {
                        hasFocus.value -= 1;
                    }
                } else if (evt.code === "ArrowDown" && hasFocus.value < (props.options.length - 1)) {
                    hasFocus.value += 1;
                    if (props.options[hasFocus.value]?.group && hasFocus.value < props.options.length) {
                        hasFocus.value += 1;
                    }
                } else if (evt.code === "Enter" && open.value) {
                    selectEvent(props.options[hasFocus.value], 'select', hasFocus.value);
                    if (!props.multiselect) {
                        closedState();
                    }
                }

                if (evt.code !== "ArrowUp" && evt.code !== "ArrowDown" && evt.code !== "Enter" && open.value) {
                    document.getElementById(searchUUID).focus();
                } else if (open.value) {
                    evt.preventDefault();
                    try {
                        var x = scrollBox.getBoundingClientRect().x;
                        var y = (document.getElementsByClassName(`elem-${hasFocus.value}`)[0].offsetHeight * hasFocus.value) - 2 * (document.getElementsByClassName(`elem-${hasFocus.value}`)[0].offsetHeight);
                        scrollBox.scrollTo(x, y);
                    } catch (error) {
                        console.log(error);
                        return;
                    }
                } else {
                    return;
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
                        // emit("select", multiSelected);
                    } else if (props.options) {
                        lastSelectedIndex.value = props.preselectIndex[0];
                        hasFocus.value = props.preselectIndex[0];
                        // emit("select", selected.value);
                    }
                }

                // Dynamically adding group item background colors
                if (props.grouped && props.groupColors?.length) {
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
                if (document.getElementById(elementID)?.contains(evt.target)) {
                    return;
                } else {
                    closedState();
                }
            });
        });

        onBeforeUnmount(() => {
            document.removeEventListener('click', () => {
                if (document.getElementById(elementID)?.contains(evt.target)) {
                    return;
                } else {
                    closedState();
                }
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
            noValueSelected,
            showDescription,
            descriptionIndex,
            lastSelectedIndex,
            hasFocus,
            searchFocus,
            multiSelectIndexes,
            searchKeyword,
            // Methods
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
