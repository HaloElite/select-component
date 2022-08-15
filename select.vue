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
        transform: translateY(10px);
        opacity: .0;
        transform-origin: bottom;
    }

    100% {
        transform: translateY(0);
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
        transform: translateY(-10px);
        opacity: .0;
        transform-origin: top;
    }

    100% {
        transform: translateY(0);
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

.focused:focus {
    border: 1px variables.$highlight solid;
}
</style>

<template>
<div :id="elementID" role="button" class="tw-relative tw-w-full tw-outline-none tw-select-none tw-border border-gray-02 tw-cursor-pointer tw-box-border tw-rounded-md focused" tabindex="0" @keydown.stop="arrowNavigation($event)">
    <span class="floating">{{ floatingLabel }}</span>
    <div class="tw-flex tw-justify-start bg-base tw-select-none color-primary sub-text tw-rounded-md" :class="{'border-error': hasError && !open, 'border-highlight': open, 'tw-bg-gray-50': deactivated}" @click="openCloseSelect($event)">
        <div class="tw-flex-grow tw-w-8/12 tw-rounded-l-sm tw-min-w-100" :class="{ 'open': open}">
            <template v-if="!multiselect">
                <div class="tw-p-2 tw-overflow-hidden tw-overflow-ellipsis">
                    <span class="tw-text-sm color-primary tw-whitespace-nowrap" :class="{'color-gray-01': (noValueSelected || deactivated)}">
                        {{ displayedValue }}
                    </span>
                </div>
            </template>
            <template v-if="multiselect">
                <div v-if="multiSelectIndices.length > 0 && !noValueSelected">
                    <template v-for="(idx, index) in multiSelectIndices" :key="idx">
                        <div v-if="index < 2" class="tw-inline-block bg-main-dark color-primary tw-text-sm tw-p-2 tw-mr-1 tw-rounded-md">
                            {{ options[idx][labelBy].length > 10 ? options[idx][labelBy].substring(0, 10 - 3) + "..." : options[idx][labelBy] }}
                            <div v-if="(multiSelectIndices.length > 1 || deletable)" @click.stop="deleteSelectedElement($event, 'select', index)" class="tw-leading-2 tw-bg-transparent tw-px-1 tw-rounded-full tw-inline-block">
                                <img src="@/assets/svg/icon-x-gray-thin.svg" class="tw-w-4 tw-h-4 tw-inline-block" alt="delete">
                            </div>
                        </div>
                        <div v-else-if="index === (multiSelectIndices.length - 1)" class="tw-inline-block color-primary tw-text-sm tw-p-2">
                            + {{ (index - 1) }}
                        </div>
                    </template>
                </div>
                <div v-else>
                    <div class="tw-p-2 tw-overflow-hidden tw-overflow-ellipsis">
                        <span class="tw-text-sm color-primary" :class="{'color-gray-01': (noValueSelected || deactivated)}">
                            {{ displayedValue }}
                        </span>
                    </div>
                </div>
            </template>
        </div>

        <div class="tw-flex tw-justify-end tw-flex-grow bg-base tw-w-8 tw-pr-2 tw-rounded-r-md" style="min-width: 50px;" :class="{'tw-bg-gray-100': deactivated}">
            <div v-if="!noValueSelected && deletable" @click.stop="deleteSelected($event, 'select')" class="tw-px-1 tw-py-1 tw-bg-white tw-flex tw-items-center">
                <img src="@/assets/svg/icon-x-gray-thin.svg" class="tw-w-4 tw-h-4 tw-inline-block" alt="delete">
            </div>

            <div class="tw-bg-white tw-flex tw-items-center tw-px-1 tw-py-1 tw-rounded-r-md">
                <img src="@/assets/svg/icon-arrow-rounded-down-gray.svg" class="tw-w-3 tw-h-3 tw-inline-block" :class="{'rotate': open, 'rotateUp': !open}" alt="open/close">
            </div>
        </div>
    </div>

    <transition :name="(openingDirection === 'up' ? 'rollup' : 'rolldown')">
        <div v-if="open" class="tw-absolute tw-left-0 tw-w-full tw-z-50 tw-text-white bg-base tw-cursor-pointer tw-border border-gray-02 tw-rounded-md" :class="{'tw-bottom-full': openingDirection === 'up', 'tw-top-full': openingDirection === 'down'}">

            <div v-if="searchable && openingDirection === 'down'" class="tw-px-4 tw-my-2">
                <div class="tw-flex tw-flex-row tw-border-b-2 border-gray-02">
                    <div class="tw-bg-white tw-flex tw-items-center tw-px-1 tw-py-1 tw-rounded-r-md">
                        <img src="@/assets/svg/icon-lens.svg" class="tw-w-3 tw-h-3 tw-inline-block" alt="search">
                    </div>
                    <div class="tw-flex-grow">
                        <input :id="searchUUID" type="text" v-model="searchKeyword" @input="searchForOption" placeholder="Liste filtern" class="revert-me input-select-search">
                    </div>
                    <div v-if="searchKeyword.length" @click.stop="quitSearch" class="tw-flex tw-justify-center tw-items-center">
                        <img src="@/assets/svg/icon-x-gray-thin.svg" class="tw-w-4 tw-h-4" alt="delete">
                    </div>
                </div>
            </div>

            <div class="tw-flex tw-flex-row">
                <div id="scrollBox" class="dropdown-params tw-flex-grow tw-overflow-y-scroll tw-overflow-x-hidden bg-base" :class="{'tw-rounded-t-md': openingDirection === 'up', 'tw-rounded-b-md': openingDirection === 'down'}">
                    <template v-if="!grouped">
                        <div v-for="(option, index) of options" :key="option.value" @click="selectEvent(option, 'select', index);">
                            <!-- Show standard dropdown-->
                            <template v-if="!currentlySearching">
                                <div @mouseover="hasFocus = index" class="tw-px-4 tw-py-2 tw-text-sm color-primary item tw-shadow-sm tw-outline-none" :class="[`elem-${index}`, {'hover-color': index === hasFocus, 'bg-gray-03': (lastSelectedIndex === index && !noValueSelected), 'bg-main-dark': multiSelectIndices.includes(index)}]" @mouseenter="descriptionIndex = index; showDescription = true;">
                                    {{ option[labelBy] }}
                                </div>
                            </template>

                            <!-- Show search results -->
                            <template v-else-if="searchResults.includes(index)">
                                <div @mouseover="searchFocus = index" class="tw-px-4 tw-py-2 tw-text-sm color-primary item tw-shadow-sm tw-outline-none" :class="[`elem-${index}`, {'hover-color': index === searchFocus, 'bg-gray-03': (lastSelectedIndex === index && !noValueSelected), 'bg-main-dark': multiSelectIndices.includes(index)}]" @mouseenter="descriptionIndex = index; showDescription = true;">
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
                                    <div @click.capture.stop class="tw-px-4 tw-py-2 tw-font-semibold tw-text-md bg-main color-highlight tw-shadow-sm" :class="[`elem-${index}`, {'hover-color': index === hasFocus}, option.classNameSpec, option.classNameGen]">
                                        {{ option[labelBy] }}
                                    </div>
                                </template>

                                <!-- Show grouped standard dropdown entries -->
                                <template v-else>
                                    <div :style="`background-color: ${optionColors[index]}`" class="tw-px-4 tw-py-2 tw-text-sm color-primary item tw-shadow-sm tw-outline-none" :class="[`elem-${index}`, {'hover-color': index === hasFocus}, option.classNameSpec, option.classNameGen, {'bg-main-dark': (lastSelectedIndex === index), 'text-blue-300': multiSelectIndices.includes(index)}]" @mouseenter="descriptionIndex = index; showDescription = true;">
                                        {{ option[labelBy] }}
                                    </div>
                                </template>
                            </template>

                            <template v-else-if="searchResults.includes(index)">
                                <!-- Show search results group headers -->
                                <template v-if="option.group">
                                    <div @click.capture.stop class="tw-px-4 tw-py-2 tw-font-semibold tw-text-md bg-main color-highlight tw-shadow-sm" :class="[`elem-${index}`, {'hover-color': index === searchFocus}, option.classNameSpec, option.classNameGen]">
                                        {{ option[labelBy] }}
                                    </div>
                                </template>

                                <!-- Show search results group entries -->
                                <template v-else>
                                    <div :style="`background-color: ${optionColors[index]}`" class="tw-px-4 tw-py-2 tw-text-sm color-primary item tw-shadow-sm tw-outline-none" :class="[`elem-${index}`, {'hover-color': index === searchFocus}, option.classNameSpec, option.classNameGen, {'bg-main-dark': (lastSelectedIndex === index), 'text-blue-300': multiSelectIndices.includes(index)}]" @mouseenter="descriptionIndex = index; showDescription = true;">
                                        {{ option[labelBy] }}
                                    </div>
                                </template>
                            </template>
                        </div>
                    </template>
                </div>

                <div v-if="optionDescription && showDescription" class="dropdown-params tw-max-w-250 tw-text-left tw-px-4 tw-py-2 tw-whitespace-normal tw-overflow-y-scroll bg-primary">
                    <p>Beschreibung:</p>
                    <p class="tw-text-sm tw-text-white">{{ options[descriptionIndex].description }}</p>
                </div>
            </div>

            <div v-if="searchable && openingDirection === 'up'" class="tw-px-4 tw-my-2">
                <div class="tw-flex tw-flex-row tw-border-t-2 border-gray-02">
                    <div class="tw-bg-white tw-flex tw-items-center tw-px-1 tw-py-1 tw-rounded-r-md">
                        <img src="@/assets/svg/icon-lens.svg" class="tw-w-3 tw-h-3 tw-inline-block" alt="search">
                    </div>
                    <div class="tw-flex-grow">
                        <input :id="searchUUID" type="text" v-model="searchKeyword" @input="searchForOption" placeholder="Liste filtern" class="revert-me input-select-search" style="pointer-event: auto;">
                    </div>
                    <div  v-if="searchKeyword.length" @click.stop="quitSearch" class="tw-flex tw-justify-center tw-items-center">
                        <img src="@/assets/svg/icon-x-gray-thin.svg" class="tw-w-4 tw-h-4" alt="delete">
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
        deactivated: {
            type: Boolean,
            required: false,
            default: false
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
            openingDirection = ref("down"),
            searchUUID = uuidv4(),
            elementID = uuidv4();


        var optionColors = reactive([]);
        var supportsPassive = false, 
            timerId = null,
            element = null;

        const closedState = () => {
            open.value = false;
            showDescription.value = false;
            currentlySearching.value = false;
            searchResults.splice(0);
            searchKeyword.value = "";
        }

        const delay = (ms) => {
            return new Promise(resolve => {
                setTimeout(resolve, ms)
            });
        } 

        const checkIfListElementIsRendered = async (className) => {
           await delay(5);
           if (document.getElementsByClassName(className)[0]) {
              return Promise.resolve(true);
           }
           return await checkIfListElementIsRendered(className);
        }

        var checkExist = setInterval(function () {
            if (document.getElementsByClassName(`elem-${lastSelectedIndex.value}`)[0]) {
                document.getElementsByClassName(`elem-${lastSelectedIndex.value}`)[0].focus();
                scrollToFocusedElement();
                clearInterval(checkExist);
            }
        }, 10);

        const openCloseSelect = async (evt, pos) => {
            if (props.deactivated) {
                return;
            }
            if (open.value) {
                closedState();
            } else {
                    open.value = true;
                    // If a select was made
                    if ((lastSelectedIndex.value || lastSelectedMultipleIndex) && props.options.length) {
                        if (!props.multiselect) {
                            hasFocus.value = lastSelectedIndex.value;
                            if (await checkIfListElementIsRendered(`elem-${lastSelectedIndex.value}`)) {
                                document.getElementsByClassName(`elem-${lastSelectedIndex.value}`)[0].focus();
                                scrollToFocusedElement();
                            }
                        } else {
                            hasFocus.value = lastSelectedMultipleIndex;
                            if (await checkIfListElementIsRendered(`elem-${lastSelectedMultipleIndex}`)) {
                                document.getElementsByClassName(`elem-${lastSelectedMultipleIndex}`)[0].focus();
                                scrollToFocusedElement();
                            }
                        }
                    } else if (props.options.length) {
                        // If no element was selected
                        if (!props.grouped) {
                            if (await checkIfListElementIsRendered(`elem-0`)) {
                                document.getElementsByClassName(`elem-0`)[0].focus();
                            }

                            if (currentlySearching.value) {
                                searchFocus.value = 0;
                            } else {
                                hasFocus.value = 0;
                            }
                        } else {
                            var elem = getFirstGroupedItem();
                            if (await checkIfListElementIsRendered(`${elem.el.classNameSpec}`)) {
                                document.getElementsByClassName(`${elem.el.classNameSpec}`)[0].focus();
                            }

                            if (currentlySearching.value) {
                                searchFocus.value = elem.idx;
                            } else {
                                hasFocus.value = elem.idx;
                            }
                        }

                    }
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
                multiSelectIndices.forEach((el, idx) => {
                    temp.push(props.options[el]);
                });
                return temp;
            } else {
                return [];
            }
        });

        var multiSelectIndices = reactive([]);
        var lastSelectedMultipleIndex = null;

        var descriptionIndex = ref(1);
        var lastSelectedIndex = ref(null);
        var hasFocus = ref(null),
            searchFocus = ref(0);

        var displayedValue = computed(() => {
            if (!props.multiselect && selected.value && selected.value[props.labelBy].length) {
                return selected.value[props.labelBy];
                // return (selected.value[props.labelBy].length > 15 ? selected.value[props.labelBy].substring(0, 15 - 3) + "..." : selected.value[props.labelBy].substring(0, 15));
            } else {
                return props.noSelectionText;
            }
        })

        const equals = (a, b) => {
            return (a.length === b.length && a.every((v, i) => v === b[i]));
        }

        // Watching to changes on single v-model values
        watch(() => props.optionIndexModel, (newval) => {
            if (typeof newval === 'number' && props.options && !props.grouped) {
                lastSelectedIndex.value = newval;
                hasFocus.value = newval;
                noValueSelected.value = false;
            } else if (typeof newval === 'number' && props.options && props.grouped) {
                lastSelectedIndex.value = determineGroupedIndex(newval);
                hasFocus.value = determineGroupedIndex(newval);
                noValueSelected.value = false;
            } else if (newval === null) {
                lastSelectedIndex.value = undefined;
                hasFocus.value = 0;
                noValueSelected.value = true;
            }
        });

        // Watching to changes on multiple v-model values /as array)
        watch(() => props.optionIndexModelMultiple, (newval) => {
            try {
                if (!props.grouped) {
                    if (newval && newval.length && props.options && !equals(multiSelectIndices, newval)) {
                        // ! Primär die Indizes kopieren
                        multiSelectIndices.splice(0);
                        for (let i = 0; i < newval.length; i++) {
                            if (typeof newval[i] == 'number' && newval[i] < props.options.length && newval[i] >= 0) {
                                multiSelectIndices.push(newval[i]);
                            }
                        }
                        noValueSelected.value = false;
                    } else if (!newval.length) {
                        multiSelectIndices.splice(0);
                        noValueSelected.value = true;
                        hasFocus.value = 0;
                    }
                } else {
                    // ! When grouped return indices of select_option WITHOUT GROUP HEADER ELEMENTS
                    var temp = [];

                    newval.forEach(el => {
                        if (typeof el == 'number') {
                            temp.push(determineGroupedIndex(el));
                        }
                    });

                    // Filtering out values that are equal to current selection-indices
                    var eq = temp.filter((el, index) => {
                        if (el !== multiSelectIndices[index]) {
                            return false;
                        } else {
                            return true;
                        }
                    });

                    // If new model-values are'nt equal to current selection
                    if (newval.length && props.options && eq.length < newval.length) {
                        multiSelectIndices.splice(0);
                        for (let i = 0; i < newval.length; i++) {
                            if (typeof newval[i] == 'number' && newval[i] < props.options.length && newval[i] >= 0) {
                                multiSelectIndices.push(determineGroupedIndex(newval[i]));
                            }
                        }
                        noValueSelected.value = false;
                    } else if (!newval.length) {
                        // Reset selection if no values were selected and reset focused value to NOT be a group header 
                        multiSelectIndices.splice(0);
                        noValueSelected.value = true;
                        if (open.value) {
                            var elem = getFirstGroupedItem();
                            document.getElementsByClassName(`${elem.el.classNameSpec}`)[0].focus();
                            if (lastSelectedMultipleIndex) {
                                hasFocus.value = lastSelectedMultipleIndex;
                            } else {
                                hasFocus.value = elem.idx;
                            }
                        }
                    }
                }
            } catch (error) {
                console.log(error);
                return;    
            }
        }, {
            deep: true
        });

        watch(() => props.deactivated, (newval) => {
            if (newval) {
                closedState();
            }
        });

        // ---------------------------------------------------------------------------------------------------- Select methods
        const selectEvent = (selectedOption, eventName, index) => {
            if (!props.multiselect) {
                if (lastSelectedIndex.value === index && !noValueSelected.value && props.deletable) {
                    noValueSelected.value = true;
                    hasFocus.value = 0;
                    searchKeyword.value = "";
                    emit('update:optionIndexModel', null);
                } else if (lastSelectedIndex.value === index && !noValueSelected.value && !props.deletable) {
                    open.value = false;
                    return;
                } else {
                    lastSelectedIndex.value = index;
                    hasFocus.value = index;
                    noValueSelected.value = false;
                    // ! When grouped return index of select_option WITHOUT GROUP HEADER ELEMENTS 
                    if (!props.grouped) {
                        emit('update:optionIndexModel', index);
                    } else if (props.grouped) {
                        var temp = index;
                        for (let i = 0; i <= index; i++) {
                            if (props.options[i]?.group === true) temp = temp - 1;
                        }
                        emit('update:optionIndexModel', temp);
                    }
                }

                try {
                    emit(eventName, selected.value);
                    open.value = false;
                } catch (error) {
                    console.log("Something went wrong:", error);
                    return;
                }
            } else if (props.multiselect) {
                lastSelectedMultipleIndex = index;
                if (multiSelectIndices.includes(index)) {
                    if (!props.deletable && multiSelectIndices.length == 1) {
                        return;
                    }
                    var position = multiSelectIndices.indexOf(index);
                    multiSelectIndices.splice(position, 1);

                    if (multiSelectIndices.length === 0) {
                        noValueSelected.value = true;
                        searchKeyword.value = "";
                        emit('update:optionIndexModelMultiple', []);
                    } else {
                        var temp = [];
                        for (let i = 0; i < multiSelectIndices.length; i++) {
                            temp.push(determineStandardIndex(multiSelectIndices[i]));
                        }
                        emit('update:optionIndexModelMultiple', temp);
                    }
                } else {
                    multiSelectIndices.push(index);

                    noValueSelected.value = false;
                    // ! When grouped return index of select_option WITHOUT GROUP HEADER ELEMENTS 
                    if (!props.grouped) {
                        emit('update:optionIndexModelMultiple', multiSelectIndices);
                    } else {
                        var temp = [];

                        multiSelectIndices.forEach(idx => {
                            temp.push(determineStandardIndex(idx));
                        });
                        emit('update:optionIndexModelMultiple', temp);
                    }
                }

                try {
                    emit(eventName, multiSelected.value, multiSelectIndices);
                } catch (error) {
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
                // noValueSelected.value = true;
                emit(eventName, null);
                emit('update:optionIndexModel', null);
                open.value = false;
            } else if (props.multiselect && multiSelectIndices.length > 0) {
                multiSelectIndices.splice(0);
                open.value = false;
                emit(eventName, []);
                emit('update:optionIndexModelMultiple', []);
            }
        }

        const deleteSelectedElement = (evt, eventName, index) => {
            if (!props.deletable && multiSelectIndices.length == 1) {
                return;
            } else if (props.multiselect && multiSelectIndices.length > 0) {
                multiSelectIndices.splice(index, 1);
                emit(eventName, multiSelected.value, multiSelectIndices);
                // ! When grouped return index of select_option WITHOUT GROUP HEADER ELEMENTS 
                if (!props.grouped) {
                    emit('update:optionIndexModelMultiple', multiSelectIndices);
                } else {
                    var temp = [];

                    multiSelectIndices.forEach(idx => {
                        temp.push(determineStandardIndex(idx));
                    });
                    emit('update:optionIndexModelMultiple', temp);
                }
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

                    var i = 0;
                    while (props.options[searchResults[i]]?.group) {
                        i++;
                    }
                    searchFocus.value = searchResults[i]
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
            var elem = document.getElementsByClassName(`elem-${hasFocus.value}`)[0];
            var x = scrollBox.getBoundingClientRect().x;
            var y = (elem.offsetHeight * hasFocus.value) - 2 * (elem.offsetHeight);
            scrollBox.scrollTo(x, y);
        }

        const arrowNavigation = (evt) => {
            if (evt.code === "Escape") {
                closedState();
            }
            if (!open.value && evt.code === "Enter") {
                openCloseSelect();
                return;
            }

            var treshold = 0;
            if (props.grouped) {
                treshold = 1;
            }

            var scrollBox = document.getElementById("scrollBox");

            if (currentlySearching.value && searchResults.length > 0) {
                if (evt.code === "ArrowUp" && searchResults.indexOf(searchFocus.value) > treshold) {
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
                if (evt.code === "ArrowUp" && hasFocus.value > treshold) {
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

        const determineGroupedIndex = (index) => {
            var options = props.options.filter(el => {
                return (!el.group);
            });

            if (index >= options.length) {
                index = options.length - 1;
            } else if (index < 0) {
                index = 0;
            }

            var element = options[index];
            var idx = 0;

            props.options.every((el, index) => {
                if (el.select_label === element.select_label) {
                    idx = index;
                    return false;
                }

                return true
            });

            return idx;
        }

        const getFirstGroupedItem = (index) => {
            var idx = 0;

            var temp = props.options.find((el, index) => {
                idx = index;
                return !el.hasOwnProperty("group");
            });

            return {
                idx: idx,
                el: temp
            };
        }

        const determineStandardIndex = (index) => {
            var options = props.options.filter(el => {
                return (!el.group);
            });

            if (index > options.length) {
                index = options.length - 1;
            } else if (index < 0) {
                index = 0;
            }

            var element = props.options[index]
            var idx = 0;

            options.every((el, index) => {
                if (el.select_label === element.select_label) {
                    idx = index;
                    return false;
                }

                return true
            });

            return idx;
        }

        onMounted(() => {
            nextTick(() => {
                if (props.preselect && (typeof props.preselectIndex[0] == 'number') && props.preselectIndex[0] !== null && props.options.length) {
                    noValueSelected.value = false;

                    if (props.multiselect) {
                        // ! When grouped return indices of select_option WITHOUT GROUP HEADER ELEMENTS 
                        if (props.grouped) {
                            for (let i = 0; i < props.preselectIndex.length; i++) {
                                if (typeof props.preselectIndex[i] == 'number' && props.preselectIndex[i] < props.options.length && props.preselectIndex[i] >= 0) {
                                    multiSelectIndices.push(determineGroupedIndex(props.preselectIndex[i]));
                                    lastSelectedMultipleIndex = i;
                                }
                            }
                        } else {
                            for (let i = 0; i < props.preselectIndex.length; i++) {
                                if (typeof props.preselectIndex[i] == 'number' && props.preselectIndex[i] < props.options.length && props.preselectIndex[i] >= 0) {
                                    multiSelectIndices.push(props.preselectIndex[i]);
                                    lastSelectedMultipleIndex = i;
                                }
                            }
                        }
                        // emit("select", multiSelected);
                    } else if (props.options) {
                        // ! When grouped return index of select_option WITHOUT GROUP HEADER ELEMENTS 
                        if (props.grouped) {
                            var options = props.options.filter(el => {
                                return (!el.group);
                            });

                            if (typeof props.preselectIndex[0] == 'number' && props.preselectIndex[0] < options.length && props.preselectIndex[0] >= 0) {
                                lastSelectedIndex.value = determineGroupedIndex(props.preselectIndex[0]);
                                hasFocus.value = determineGroupedIndex(props.preselectIndex[0]);
                            }
                        } else {
                            if (typeof props.preselectIndex[0] == 'number' && props.preselectIndex[0] < props.options.length && props.preselectIndex[0] >= 0) {
                                lastSelectedIndex.value = props.preselectIndex[0];
                                hasFocus.value = props.preselectIndex[0];
                            }
                        }
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


            // Event Listener and calculations for opening direction
            element = document.getElementById(elementID);

            if ((window.innerHeight - element.getBoundingClientRect().bottom) > 250) {
                openingDirection.value = "down";
            } else {
                openingDirection.value = "up";
            }


            try {
                var opts = Object.defineProperty({}, 'passive', {
                    get: function() {
                        supportsPassive = true;
                    }
                });
                window.addEventListener("testPassive", null, opts);
                window.removeEventListener("testPassive", null, opts);
            } catch (e) {}

            window.addEventListener("scroll", () => {
                if (timerId) return;
                timerId  =  setTimeout(function () {
                    timerId  =  null;

                    if (element) {
                        if ((window.innerHeight - element.getBoundingClientRect().bottom) > 250) {
                            openingDirection.value = "down";
                        } else {
                            openingDirection.value = "up";
                        }             
                    }
                }, 100);
            }, supportsPassive ? { passive: true } : false);
        });

        onBeforeUnmount(() => {
            document.removeEventListener('click', () => {
                if (document.getElementById(elementID)?.contains(evt.target)) {
                    return;
                } else {
                    closedState();
                }
            });

            window.removeEventListener("scroll", () => {
                if (timerId) return;
                timerId  =  setTimeout(function () {
                    console.log("CALLED");
                    timerId  =  null;

                    if (element) {
                        if ((window.innerHeight - element.getBoundingClientRect().bottom) > 250) {
                            openingDirection.value = "down";
                        } else {
                            openingDirection.value = "up";
                        }             
                    }
                }, 100);
            }, supportsPassive ? { passive: true } : false);
        });

        return {
            openingDirection,
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
            multiSelectIndices,
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
