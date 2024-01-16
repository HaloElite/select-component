<style lang="scss" scoped>
@use '~/styles/components/_dropdownselect';
</style>

<template>
    <div :ref="(el) => { containerElement = el }" role="button"
        class="pk-select tw-relative tw-w-full tw-outline-none tw-select-none tw-cursor-pointer tw-rounded-md tw-box-border"
        :tabindex="tabindex" @keydown.stop="arrowNavigation($event)">
        <span class="floating">{{ floatingLabel }}</span>

        <div class="tw-flex tw-h-8 tw-justify-start tw-items-center bg-base tw-select-none color-primary sub-text tw-rounded-md tw-border tw-box-border pk-select-inner"
            :class="{ 'border-gray-02': !rimless && !activeElement, 'border-primary': activeElement, 'tw-border-transparent': rimless, 'border-error': hasError && !open, 'border-highlight': open, 'tw-bg-gray-50': deactivated }"
            @click="openCloseSelect($event)">
            <div class="tw-w-full tw-rounded-l-sm tw-min-w-50" :class="{ 'open': open }">
                <template v-if="!multiselect">
                    <div class="tw-pl-2 tw-overflow-hidden tw-overflow-ellipsis" :class="{ 'tw-pl-10': hasIcon }">
                        <img v-if="hasIcon && !deactivated" :src="iconPath" alt="icon" width="20px" height="20px"
                            class="tw-absolute tw-left-2 tw-top-1/2 tw-transform -tw-translate-y-1/2 tw-h-auto tw-w-5"
                            @dragstart.prevent>
                        <img v-if="hasIcon && deactivated" :src="iconPathDeactivated" alt="icon" width="20px" height="20px"
                            class="tw-absolute tw-left-2 tw-top-1/2 tw-transform -tw-translate-y-1/2 tw-h-auto tw-w-5"
                            @dragstart.prevent>

                        <span class="tw-text-sm color-primary tw-whitespace-nowrap"
                            :class="{ 'color-gray-01': (noValueSelected || deactivated), 'tw-flex tw-justify-between tw-items-center': showAdditionalInfoDisplayed && additionalInfoDisplayed.length }">
                            <span
                                :class="{ 'tw-w-3/4 tw-min-w-50': showAdditionalInfoDisplayed, 'tw-w-full': !showAdditionalInfoDisplayed }"
                                class="tw-text-ellipsis tw-overflow-hidden tw-whitespace-nowrap">
                                {{ displayedValue }}
                            </span>
                            <span v-if="showAdditionalInfoDisplayed && additionalInfoDisplayed.length && options.length"
                                :title="additionalInfoDisplayed"
                                class="tw-overflow-hidden tw-overflow-ellipsis tw-whitespace-nowrap tw-max-w-100 tw-border tw-px-1 tw-rounded-sm border-gray-02 color-gray-01">
                                {{ additionalInfoDisplayed }}
                            </span>
                        </span>
                    </div>
                </template>
                <template v-if="multiselect">
                    <div v-if="multiSelectIndices.length > 0 && !noValueSelected"
                        class="tw-flex tw-justify-start tw-items-center tw-overflow-hidden">
                        <template v-for="(idx, index) in multiSelectIndices" :key="idx">
                            <div v-if="index < 2"
                                class="tw-flex tw-flex-nowrap tw-items-center tw-whitespace-nowrap bg-main-dark color-primary tw-text-sm tw-px-2 tw-mr-1 tw-rounded-md"
                                :class="{ 'tw-ml-1': idx === 0 }">
                                {{ options[idx][labelBy].length > 10 ? options[idx][labelBy].substring(0, 10 - 3) + "..." :
                                    options[idx][labelBy] }}
                                <div v-if="(multiSelectIndices.length > 1 || deletable)"
                                    @click.stop="deleteSelectedElement($event, 'select', index)"
                                    class="tw-w-4 tw-h-4 tw-leading-2 tw-bg-transparent tw-flex tw-justify-start tw-items-center tw-ml-1 tw-rounded-full">
                                    <img src="@/assets/svg/icon-xr-primary.svg" class="tw-h-4 tw-w-4 tw-shrink-0"
                                        alt="delete" @dragstart.prevent>
                                </div>
                            </div>
                            <div v-else-if="index === (multiSelectIndices.length - 1)"
                                class="tw-inline-block color-primary tw-text-sm tw-p-2">
                                + {{ (index - 1) }}
                            </div>
                        </template>
                    </div>
                    <div v-else>
                        <div class="tw-px-2 tw-overflow-hidden tw-overflow-ellipsis" :class="{ 'tw-pl-10': hasIcon }">
                            <img v-if="hasIcon && !deactivated" :src="iconPath" alt="icon" width="20px" height="20px"
                                class="tw-absolute tw-left-2 tw-top-1/2 tw-transform -tw-translate-y-1/2 tw-h-auto tw-w-5">
                            <img v-if="hasIcon && deactivated" :src="iconPathDeactivated" alt="icon" width="20px"
                                height="20px"
                                class="tw-absolute tw-left-2 tw-top-1/2 tw-transform -tw-translate-y-1/2 tw-h-auto tw-w-5">

                            <span class="tw-text-sm color-primary"
                                :class="{ 'color-gray-01': (noValueSelected || deactivated) }">
                                {{ displayedValue }}
                            </span>
                        </div>
                    </div>
                </template>
            </div>

            <div class="tw-flex tw-justify-end tw-flex-grow bg-base tw-pr-1 tw-rounded-r-md"
                :class="{ 'tw-bg-gray-100': deactivated }">
                <div v-if="!noValueSelected && deletable" @click.stop="deleteSelected($event, 'select')"
                    class="tw-bg-white tw-flex tw-items-center tw-w-4 tw-h-4 tw-shrink-0">
                    <img src="@/assets/svg/icon-xr-red.svg" class="delete-xr-red tw-w-4 tw-h-4" alt="delete">
                </div>

                <div class="tw-bg-white tw-flex tw-items-center tw-rounded-r-md tw-w-4 tw-h-4 tw-shrink-0"
                    :class="{ 'tw-invisible': hideChevron }">
                    <img v-if="!deactivated" src="@/assets/svg/icon-chevron-down-primary.svg"
                        class="tw-w-4 tw-h-4 tw-inline-block tw-shrink-0"
                        :class="{ 'rotate': open && enableChevronAnimation, 'rotateUp': !open && enableChevronAnimation }"
                        alt="open/close" @dragstart.prevent>
                    <img v-if="deactivated" src="@/assets/svg/icon-chevron-down-gray.svg"
                        class="tw-w-4 tw-h-4 tw-inline-block tw-shrink-0"
                        :class="{ 'rotate': open && enableChevronAnimation, 'rotateUp': !open && enableChevronAnimation }"
                        alt="open/close" @dragstart.prevent>
                </div>
            </div>
        </div>

        <transition :name="(openingDirection === 'up' ? 'rollup' : 'rolldown')">
            <div v-if="open"
                class="tw-absolute tw-left-0 tw-z-50 tw-text-white bg-base tw-cursor-pointer tw-box-border tw-rounded-md"
                :class="{ 'tw-w-full': !flexibleWidth, 'tw-w-max tw-min-w-full': flexibleWidth, 'tw-bottom-full tw-transform -tw-translate-y-1 shadow-elevate-black': openingDirection === 'up', 'tw-top-full tw-transform tw-translate-y-1 shadow-elevate-black': openingDirection === 'down' }">

                <div v-if="searchable && openingDirection === 'down' && options.length" class="tw-px-4 tw-my-2">
                    <div class="tw-flex tw-flex-row tw-border-b border-gray-02">
                        <div class="tw-bg-white tw-flex tw-items-center tw-px-1 tw-py-1 tw-rounded-r-md">
                            <img src="@/assets/svg/icon-lens.svg" class="tw-w-3 tw-h-3 tw-inline-block" alt="search"
                                @dragstart.prevent>
                        </div>
                        <div class="tw-flex-grow">
                            <input :ref="(el) => { searchInput = el }" type="text" v-model="searchKeyword"
                                @input="searchForOption" placeholder="Liste filtern" class="revert-me input-select-search">
                        </div>
                        <div v-if="searchKeyword.length" @click.stop="quitSearch"
                            class="tw-flex tw-justify-center tw-items-center">
                            <img src="@/assets/svg/icon-xr-primary.svg" class="tw-w-4 tw-h-4" alt="delete"
                                @dragstart.prevent>
                        </div>
                    </div>
                </div>

                <div class="tw-flex tw-flex-row" :ref="(el) => { dropdownTab = el }">
                    <div id="scrollBox" ref="scrollBox"
                        class="dropdown-params tw-flex-grow tw-overflow-y-auto tw-overflow-x-hidden bg-base tw-break-words"
                        :class="{ 'tw-rounded-t-md': openingDirection === 'up', 'tw-rounded-b-md': openingDirection === 'down' && !addElementActive }">

                        <!-- ############# NOT GROUPED SELECT OPTIONS ############# -->
                        <template v-if="!grouped">
                            <div v-for="(option, index) of  options " :key="option.value"
                                @click="selectEvent(option, 'select', index);">
                                <!-- Show standard dropdown-->
                                <template v-if="!currentlySearching">
                                    <div @mouseover=" hasFocus = index"
                                        class="tw-px-4 tw-py-2 tw-text-sm color-primary tw-shadow-sm tw-outline-none"
                                        :ref="setElemRefs"
                                        :class="`dropdown-item-${hoverColor}`, [{ 'select-hover-highlight': index === hasFocus && hoverColor === 'highlight', 'select-hover-gray': index === hasFocus && hoverColor === 'gray', 'bg-gray-03': (lastSelectedIndex === index && lastSelectedIndex === hasFocus && !noValueSelected), 'bg-main-dark': multiSelectIndices.includes(index) }]"
                                        @mouseenter=" descriptionIndex = index; showDescription = true;">
                                        <span class="tw-inline-flex tw-items-center">
                                            {{ option[labelBy] }}
                                            <div v-if="showIndicator" :title="option.indicatorTitle"
                                                class="tw-inline-block tw-rounded-full tw-p-1 tw-border-2 tw-ml-1"
                                                :class="{ 'bg-success border-success': option.indicatorFilled, 'bg-base border-success': !option.indicatorFilled }">
                                            </div>
                                        </span>
                                        <PkSelectAdditionals v-if="displayAdditionals" :info1="additionalInfo[index]?.info1"
                                            :info2="additionalInfo[index]?.info2" :hasFocus="index === hasFocus">
                                        </PkSelectAdditionals>
                                    </div>
                                </template>

                                <!-- Show search results -->
                                <template v-else-if="searchResults.includes(index)">
                                    <div @mouseover=" searchFocus = index"
                                        class="tw-px-4 tw-py-2 tw-text-sm color-primary tw-shadow-sm tw-outline-none"
                                        :ref="setElemRefs"
                                        :class="`dropdown-item-${hoverColor}`, [{ 'select-hover-highlight': index === searchFocus && hoverColor === 'highlight', 'select-hover-gray': index === searchFocus && hoverColor === 'gray', 'bg-gray-03': (lastSelectedIndex === index && searchFocus === lastSelectedIndex && !noValueSelected), 'bg-main-dark': multiSelectIndices.includes(index) }]"
                                        @mouseenter=" descriptionIndex = index; showDescription = true;">
                                        <span class="tw-inline-flex tw-items-center">
                                            {{ option[labelBy] }}
                                            <div v-if="showIndicator" :title="option.indicatorTitle"
                                                class="tw-inline-block tw-rounded-full tw-p-1 tw-border-2 tw-ml-1"
                                                :class="{ 'bg-success border-success': option.indicatorFilled, 'bg-base border-success': !option.indicatorFilled }">
                                            </div>
                                        </span>
                                        <PkSelectAdditionals v-if="displayAdditionals" :info1="additionalInfo[index]?.info1"
                                            :info2="additionalInfo[index]?.info2" :hasFocus="index === searchFocus">
                                        </PkSelectAdditionals>
                                    </div>
                                </template>
                            </div>
                        </template>

                        <!-- ############# GROUPED SELECT OPTIONS ############# -->
                        <template v-if="grouped">
                            <div v-for="( option, index ) of  options " :key="option.value"
                                @click=" selectEvent(option, 'select', index);">
                                <template v-if="!currentlySearching">
                                    <!-- Show grouped standard dropdown headers -->
                                    <template v-if="option.group">
                                        <div @click.capture.stop
                                            class="tw-px-4 tw-py-2 tw-font-semibold tw-text-md bg-main color-highlight tw-shadow-sm"
                                            :ref="setElemRefs"
                                            :class="[{ 'select-hover-highlight': index === hasFocus && hoverColor === 'highlight', 'select-hover-gray': index === hasFocus && hoverColor === 'gray' }, option.classNameSpec, option.classNameGen]">
                                            <span class="tw-inline-flex tw-items-center">{{ option[labelBy] }}
                                                <div v-if="showIndicator" :title="option.indicatorTitle"
                                                    class="tw-inline-block tw-rounded-full tw-p-1 tw-border-2 tw-ml-1"
                                                    :class="{ 'bg-success border-success': option.indicatorFilled, 'bg-base border-success': !option.indicatorFilled }">
                                                </div>
                                            </span>
                                        </div>
                                    </template>

                                    <!-- Show grouped standard dropdown entries -->
                                    <template v-else>
                                        <div :style="`background-color: ${optionColors[index]}`"
                                            class="tw-px-4 tw-py-2 tw-text-sm color-primary tw-shadow-sm tw-outline-none"
                                            :ref="setElemRefs"
                                            :class="`dropdown-item-${hoverColor}`, [{ 'select-hover-highlight': index === hasFocus && hoverColor === 'highlight', 'select-hover-gray': index === hasFocus && hoverColor === 'gray' }, option.classNameSpec, option.classNameGen, { 'bg-main-dark': (lastSelectedIndex === index), 'text-blue-300': multiSelectIndices.includes(index) }]"
                                            @mouseenter=" descriptionIndex = index; showDescription = true;">
                                            <span class="tw-inline-flex tw-items-center">{{ option[labelBy] }}
                                                <div v-if="showIndicator" :title="option.indicatorTitle"
                                                    class="tw-inline-block tw-rounded-full tw-p-1 tw-border-2 tw-ml-1"
                                                    :class="{ 'bg-success border-success': option.indicatorFilled, 'bg-base border-success': !option.indicatorFilled }">
                                                </div>
                                            </span>
                                        </div>
                                    </template>
                                </template>

                                <template v-else-if="searchResults.includes(index)">
                                    <!-- Show search results group headers -->
                                    <template v-if="option.group">
                                        <div @click.capture.stop
                                            class="tw-px-4 tw-py-2 tw-font-semibold tw-text-md bg-main color-highlight tw-shadow-sm"
                                            :ref="setElemRefs"
                                            :class="[{ 'select-hover-highlight': index === searchFocus && hoverColor === 'highlight', 'select-hover-gray': index === searchFocus && hoverColor === 'gray' }, option.classNameSpec, option.classNameGen]">
                                            {{ option[labelBy] }}
                                        </div>
                                    </template>

                                    <!-- Show search results group entries -->
                                    <template v-else>
                                        <div :style="`background-color: ${optionColors[index]}`"
                                            class="tw-px-4 tw-py-2 tw-text-sm color-primary tw-shadow-sm tw-outline-none"
                                            :ref="setElemRefs"
                                            :class="`dropdown-item-${hoverColor}`, [{ 'select-hover-highlight': index === searchFocus && hoverColor === 'highlight', 'select-hover-gray': index === searchFocus && hoverColor === 'gray' }, option.classNameSpec, option.classNameGen, { 'bg-main-dark': (lastSelectedIndex === index), 'text-blue-300': multiSelectIndices.includes(index) }]"
                                            @mouseenter=" descriptionIndex = index; showDescription = true;">
                                            {{ option[labelBy] }}
                                        </div>
                                    </template>
                                </template>
                            </div>
                        </template>
                    </div>

                    <div v-if="optionDescription && showDescription"
                        class="tw-absolute tw-rounded-md tw-left-full dropdown-params tw-max-w-250 tw-text-left tw-px-4 tw-py-2 tw-whitespace-normal tw-overflow-y-auto bg-primary">
                        <p class="tw-pb-2">Beschreibung:</p>
                        <p class="tw-text-sm tw-text-white">{{ options[descriptionIndex].description }}</p>
                    </div>
                </div>

                <div v-if="searchable && openingDirection === 'up' && options.length" class="tw-px-4 tw-my-2">
                    <div class="tw-flex tw-flex-row tw-border-t border-gray-02">
                        <div class="tw-bg-white tw-flex tw-items-center tw-px-1 tw-py-1 tw-rounded-r-md">
                            <img src="@/assets/svg/icon-lens.svg" class="tw-w-3 tw-h-3 tw-inline-block" alt="search"
                                @dragstart.prevent>
                        </div>
                        <div class="tw-flex-grow">
                            <input :ref="(el) => { searchInput = el }" type="text" v-model="searchKeyword"
                                @input="searchForOption" placeholder="Liste filtern" class="revert-me input-select-search"
                                style="pointer-event: auto;">
                        </div>
                        <div v-if="searchKeyword.length" @click.stop="quitSearch"
                            class="tw-flex tw-justify-center tw-items-center">
                            <img src="@/assets/svg/icon-xr-primary.svg" class="tw-w-4 tw-h-4" alt="delete"
                                @dragstart.prevent>
                        </div>
                    </div>
                </div>

                <PkSelectAddElement class="tw-rounded-b-md" :class="{ 'tw-rounded-t-md': options.length === 0 }"
                    v-if="addElementActive" :text="addElementText" @addElement="addElement"></PkSelectAddElement>
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
    onBeforeUpdate,
    nextTick
} from 'vue'

import PkSelectAdditionals from './PkSelectAdditionals.vue';
import PkSelectAddElement from './PkSelectAddElement.vue';

export default {
    name: "pk_select",
    props: {
        tabindex: {
            type: Number,
            default: 0
        },
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
        activeElement: {
            type: Boolean,
            required: false,
            default: false
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
        },
        rimless: {
            type: Boolean,
            required: false,
            default: false
        },
        hasIcon: {
            type: Boolean,
            required: false,
            default: false
        },
        iconPath: {
            type: String,
            default: "",
            required: false
        },
        iconPathDeactivated: {
            type: String,
            default: "",
            required: false
        },
        displayAdditionals: {
            type: Boolean,
            required: false,
            default: false
        },
        additionalInfo: {
            type: Array,
            required: false
        },
        addElementText: {
            type: String,
            required: false
        },
        addElementActive: {
            type: Boolean,
            required: false,
            default: false
        },
        showAdditionalInfoDisplayed: {
            type: Boolean,
            required: false,
            default: false
        },
        additionalInfoDisplayed: {
            type: String,
            default: ""
        },
        showIndicator: {
            type: Boolean,
            required: false,
            default: false
        },
        flexibleWidth: {
            type: Boolean,
            required: false,
            default: false
        },
        hoverColor: {
            type: String,
            required: false,
            default: "highlight"
        },
        parentElement: {
            type: String,
            default: ""
        },
        useParentScroll: {
            type: Boolean,
            default: false
        },
        hideChevron: {
            type: Boolean,
            default: false
        }
    },
    components: {
        PkSelectAdditionals,
        PkSelectAddElement
    },
    emits: ['select', 'emitSearchString', 'addElement', ['update:optionIndexModel'],
        ['update:optionIndexModelMultiple']
    ],
    setup(props, {
        emit
    }) {
        // Template refs
        const scrollBox = ref(null),
            elems = ref([]),
            dropdownTab = ref(null),
            containerElement = ref(null),
            searchInput = ref(null);

        const setElemRefs = (el) => {
            if (el) {
                elems.value.push(el);
            }
        };

        const controller = new AbortController();
        // DROPDOWN CONFIG
        var open = ref(false),
            noValueSelected = ref(true),
            showDescription = ref(false),
            openingDirection = ref("down");

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

        const checkIfListElementIsRendered = async () => {
            await delay(5);

            if (elems.value.length) {
                return Promise.resolve(true);
            }
            return await checkIfListElementIsRendered();
        }

        const openCloseSelect = async (evt, pos) => {
            if (!enableChevronAnimation.value) enableChevronAnimation.value = true;
            if (props.deactivated) {
                return;
            }
            if (open.value) {
                closedState();
            } else {
                if (!props.options.length) {
                    return;
                }
                open.value = true;
                // If a select was made
                if ((lastSelectedIndex.value || lastSelectedMultipleIndex) && props.options && props.options.length) {
                    if (await checkIfListElementIsRendered()) {
                        if (!props.multiselect) {
                            hasFocus.value = lastSelectedIndex.value;
                            elems.value[0].focus();
                            scrollToFocusedElement();
                        } else {
                            hasFocus.value = lastSelectedMultipleIndex;
                            elems.value[lastSelectedMultipleIndex].focus()
                            scrollToFocusedElement();
                        }
                    }
                } else if (props.options && props.options.length) {
                    // If no element was selected
                    if (!props.grouped) {
                        if (await checkIfListElementIsRendered()) {
                            elems.value[0].focus();
                        }

                        if (currentlySearching.value) {
                            searchFocus.value = 0;
                        } else {
                            hasFocus.value = 0;
                        }
                    } else {
                        var elem = getFirstGroupedItem();
                        if (await checkIfListElementIsRendered()) {
                            const entryToFocus = elems.value.find((element) => element.classList.contains(elem.el.classNameSpec));
                            entryToFocus.focus();
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
                if (!props.options.length) {
                    return "Keine Optionen verfügbar"
                } else {
                    return props.noSelectionText;
                }
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
                            const entryToFocus = elems.value.find((element) => element.classList.contains(elem.el.classNameSpec));
                            entryToFocus.focus();

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
                    // noValueSelected.value = true;
                    // hasFocus.value = 0;
                    // searchKeyword.value = "";
                    // emit('update:optionIndexModel', null);
                    openCloseSelect();
                    return;
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
                    openCloseSelect();
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
            emit('emitSearchString', searchKeyword.value);

            currentlySearching.value = true;
            searchResults.splice(0);

            var searchString = searchKeyword.value.toLowerCase();

            nextTick(() => {
                if (searchString.length > 0) {
                    if (!props.grouped) {
                        props.options.forEach((element, index) => {
                            if (!element.hasOwnProperty('group') && element[props.labelBy]) {
                                if (element[props.labelBy].toLowerCase().indexOf(searchString) > -1) {
                                    searchResults.push(index);
                                }
                            }
                        });
                        searchFocus.value = searchResults[0];
                    } else if (props.grouped) {
                        props.options.forEach((item, index) => {
                            if (!item?.group && item[props.labelBy].toLowerCase().indexOf(searchString) > -1) {
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
            })
        }

        const quitSearch = () => {
            currentlySearching.value = false;
            searchKeyword.value = "";
            searchResults.splice(0);
        }

        // ---------------------------------------------------------------------------------------------------- Arrow navigation
        const scrollToFocusedElement = () => {
            var elem = elems.value[hasFocus.value],
                x = scrollBox.value?.getBoundingClientRect().x,
                y = (elem?.offsetHeight * hasFocus.value) - 2 * (elem?.offsetHeight);

            scrollBox.value?.scrollTo(x, y);
        }

        const focusedIsGroupHeader = (searching, direction, backupfocus) => {
            if (searching) {
                if (props.options[searchFocus.value].hasOwnProperty("group")) {

                    if (direction === "up" && searchResults.indexOf(searchFocus.value) > 0) {
                        let filtered = searchResults.filter((el) => {
                            if (el < searchFocus.value && !props.options[el].hasOwnProperty("group")) {
                                return el;
                            }
                        });

                        if (filtered.length) {
                            searchFocus.value = filtered[filtered.length - 1];
                            return true;
                        } else {
                            searchFocus.value = backupfocus;
                            return false;
                        }
                    } else if (direction === "down" && searchResults.indexOf(searchFocus.value) < (searchResults.length - 1)) {
                        let filtered = searchResults.filter((el) => {
                            if (!props.options[el].hasOwnProperty("group")) {
                                return el;
                            }
                        });

                        if (filtered.length > 0 && searchFocus.value <= filtered[filtered.length - 1]) {
                            searchFocus.value = filtered.find((e) => e > searchFocus.value);
                            return true;
                        } else if (filtered.length > 0) {
                            searchFocus.value = filtered[filtered.length - 1];
                            return true;
                        } else {
                            searchFocus.value = backupfocus;
                            return false;
                        }
                    } else {
                        return false;
                    }
                } else {
                    return true;
                }
            } else if (!searching) {
                if (props.options[hasFocus.value].hasOwnProperty("group")) {
                    if (direction === "up" && hasFocus.value > 0) {
                        while (props.options[hasFocus.value].hasOwnProperty("group")) {
                            if (hasFocus.value > 0) {
                                hasFocus.value--;
                            } else {
                                hasFocus.value = backupfocus;
                                break;
                            }
                        }

                        return true;
                    } else if (direction === "down" && hasFocus.value < (props.options.length - 1)) {
                        while (props.options[hasFocus.value].hasOwnProperty("group")) {
                            if (hasFocus.value < props.options.length - 1) {
                                hasFocus.value++;
                            } else {
                                hasFocus.value = backupfocus;
                                break;
                            }
                        }

                        return true;
                    } else {
                        return false;
                    }
                } else {
                    return true;
                }
            }
        };

        const checkGroupedResults = () => {
            let filtered = searchResults.filter((el) => {
                if (!props.options[el].hasOwnProperty("group")) {
                    return el;
                }
            });

            if (filtered.length) {
                return true;
            } else {
                return false;
            }
        }

        const checkGroupedOptions = () => {
            let filtered = props.options.filter((el) => {
                if (!el.hasOwnProperty("group")) {
                    return el;
                }
            });

            if (filtered.length) {
                return true;
            } else {
                return false;
            }
        }

        const arrowNavigation = (evt) => {
            if (evt.code === "Tab") return;

            if (!checkGroupedOptions()) return;

            if (evt.code === "Escape") closedState();

            if (props.optionDescription) showDescription.value = true;

            if (!open.value && evt.code === "Enter") {
                openCloseSelect();
                return;
            }

            var treshold = 0;
            if (props.grouped) {
                treshold = 1;
            }

            if (currentlySearching.value && searchResults.length > 0) {
                if (props.grouped && (evt.code !== "ArrowUp" || evt.code !== "ArrowDown" || evt.code !== "Enter")) {
                    if (!checkGroupedResults()) return;
                }

                if (evt.code === "ArrowUp" && searchResults.indexOf(searchFocus.value) > treshold) {
                    var temp = searchFocus.value;
                    searchFocus.value = searchResults[(searchResults.indexOf(searchFocus.value) - 1)];
                    if (props.grouped) {
                        if (!focusedIsGroupHeader(true, "up", temp)) return;
                    }
                } else if (evt.code === "ArrowDown" && searchResults.indexOf(searchFocus.value) < (searchResults.length - 1)) {
                    var temp = searchFocus.value;
                    searchFocus.value = searchResults[(searchResults.indexOf(searchFocus.value) + 1)];
                    if (props.grouped) {
                        if (!focusedIsGroupHeader(true, "down", temp)) return;
                    }
                } else if (evt.code === "Enter" && open.value) {
                    evt.preventDefault();
                    selectEvent(props.options[searchFocus.value], 'select', searchFocus.value);
                    if (!props.multiselect) {
                        closedState();
                    }
                }

                if (props.optionDescription) {
                    descriptionIndex.value = searchFocus.value;
                }

                if (evt.code !== "ArrowUp" && evt.code !== "ArrowDown" && evt.code !== "Enter" && open.value) {
                    searchInput.value?.focus();
                } else if (open.value) {
                    evt.preventDefault();
                    try {
                        var x = scrollBox.value?.getBoundingClientRect().x;
                        var y = elems.value[searchFocus.value]?.offsetTop + elems.value[searchFocus.value]?.offsetHeight / 2 - dropdownTab.value?.offsetHeight / 2;

                        scrollBox.value?.scrollTo(x, y);
                    } catch (error) {
                        console.log(error);
                        return;
                    }
                } else {
                    return;
                }
            } else {
                if (evt.code === "ArrowUp" && hasFocus.value > treshold) {
                    var temp = searchFocus.value;
                    hasFocus.value -= 1;
                    if (props.grouped) {
                        if (!focusedIsGroupHeader(false, "up", temp)) return;
                    }
                } else if (evt.code === "ArrowDown" && hasFocus.value < (props.options.length - 1)) {
                    var temp = searchFocus.value;
                    hasFocus.value += 1;
                    if (props.grouped) {
                        if (!focusedIsGroupHeader(false, "down", temp)) return;
                    }
                } else if (evt.code === "Enter" && open.value) {
                    evt.preventDefault();
                    selectEvent(props.options[hasFocus.value], 'select', hasFocus.value);
                    if (!props.multiselect) {
                        closedState();
                    }
                }

                if (props.optionDescription) {
                    descriptionIndex.value = hasFocus.value;
                }

                if (evt.code !== "ArrowUp" && evt.code !== "ArrowDown" && evt.code !== "Enter" && open.value) {
                    searchInput.value?.focus();
                } else if (open.value) {
                    evt.preventDefault();
                    try {
                        var x = scrollBox.value?.getBoundingClientRect().x;
                        var y = elems.value[hasFocus.value]?.offsetTop + elems.value[hasFocus.value]?.offsetHeight / 2 - dropdownTab.value?.offsetHeight / 2;

                        scrollBox.value?.scrollTo(x, y);
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
                // el[props.labelBy] working if no double labels are included
                if (el.select_value === element.select_value) {
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
                // el[props.labelBy] working if no double labels are included
                if (el.select_value === element.select_value) {
                    idx = index;
                    return false;
                }

                return true
            });

            return idx;
        }

        const addElement = () => {
            openCloseSelect();
            emit('addElement')
        }

        let enableChevronAnimation = ref(false);

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
                    var headers = 0,
                        members = [],
                        memberVal = 0,
                        section = -1;

                    props.options.forEach(el => {
                        if (el.group) {
                            headers++;
                            section++;
                            memberVal = 0;
                        } else if (!el.group) {
                            memberVal++;
                            members[section] = memberVal;
                        }
                    });

                    for (var index = 0; index <= headers; index++) {
                        for (let idx = 0; idx <= members[index]; idx++) {
                            optionColors.push(props.groupColors[index]);
                        }
                    }
                }
            });

            window.addEventListener('mouseup', (evt) => {
                if (containerElement.value?.contains(evt.composedPath()[0])) {
                    return;
                } else {
                    closedState();
                }
            }, {
                signal: controller.signal
            });

            // Event Listener and calculations for opening direction
            if (!props.useParentScroll) {
                if ((window.innerHeight - containerElement.value?.getBoundingClientRect().bottom) > 250) {
                    openingDirection.value = "down";
                } else {
                    openingDirection.value = "up";
                }
            } else {
                let dist = (props.parentElement?.getBoundingClientRect().height - containerElement.value?.getBoundingClientRect().bottom);

                if (dist > 200) {
                    openingDirection.value = "down";
                } else {
                    openingDirection.value = "up";
                }
            }

            try {
                var opts = Object.defineProperty({}, 'passive', {
                    get: function () {
                        supportsPassive = true;
                    }
                });
                window.addEventListener("testPassive", null, opts);
                window.removeEventListener("testPassive", null, opts);
            } catch (e) { }

            if (!props.useParentScroll) {
                window.addEventListener("scroll", () => {
                    if (timerId) return;
                    timerId = setTimeout(function () {
                        timerId = null;

                        if (containerElement.value) {
                            let dist = (window.innerHeight - containerElement.value?.getBoundingClientRect().bottom);
                            if (dist > 250) {
                                openingDirection.value = "down";
                            } else {
                                openingDirection.value = "up";
                            }
                        }
                    }, 100);
                }, {
                    signal: controller.signal,
                    passive: supportsPassive ? true : false
                });
            } else {
                props.parentElement.addEventListener("scroll", () => {
                    if (timerId) return;
                    timerId = setTimeout(function () {
                        timerId = null;

                        if (containerElement.value) {
                            let dist = (props.parentElement?.getBoundingClientRect().bottom - containerElement.value?.getBoundingClientRect().bottom);

                            if (dist > 275) {
                                openingDirection.value = "down";
                            } else {
                                openingDirection.value = "up";
                            }
                        }
                    }, 100);
                }, {
                    signal: controller.signal,
                    passive: supportsPassive ? true : false
                });
            }
        });

        onBeforeUnmount(() => {
            controller.abort();
        });

        // Make sure to reset the refs before each update.
        onBeforeUpdate(() => {
            elems.value = [];
        });

        return {
            openingDirection,
            displayedValue,
            selected,
            multiSelected,
            optionColors,
            searchResults,
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
            enableChevronAnimation,
            // Methods
            openCloseSelect,
            selectEvent,
            deleteSelected,
            deleteSelectedElement,
            searchForOption,
            quitSearch,
            arrowNavigation,
            addElement,
            // Template refs
            scrollBox,
            elems,
            dropdownTab,
            containerElement,
            searchInput,
            setElemRefs
        }
    }
};
</script>
