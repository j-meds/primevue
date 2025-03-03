<template>
    <div ref="container" :id="id" :class="containerClass" @click="onContainerClick">
        <input v-if="editable" ref="focusInput" :id="inputId" type="text" :style="inputStyle" :class="inputStyleClass" :value="editableInputValue" :placeholder="placeholder" :tabindex="!disabled ? tabindex : -1" :disabled="disabled" autocomplete="off"
            role="combobox" :aria-label="ariaLabel" :aria-labelledby="ariaLabelledby" aria-haspopup="listbox" :aria-expanded="overlayVisible" :aria-controls="id + '_list'" :aria-activedescendant="focused ? focusedOptionId : undefined"
            @focus="onFocus" @blur="onBlur" @keydown="onKeyDown" @input="onEditableInput" v-bind="inputProps">
        <span v-else ref="focusInput" :id="inputId" :style="inputStyle" :class="inputStyleClass" :tabindex="!disabled ? tabindex : -1" role="combobox" :aria-label="ariaLabel || (label === 'p-emptylabel' ? undefined : label)" :aria-labelledby="ariaLabelledby"
            aria-haspopup="listbox" :aria-expanded="overlayVisible" :aria-controls="id + '_list'" :aria-activedescendant="focused ? focusedOptionId : undefined" :aria-disabled="disabled"
            @focus="onFocus" @blur="onBlur" @keydown="onKeyDown" v-bind="inputProps">
            <slot name="value" :value="modelValue" :placeholder="placeholder">{{label === 'p-emptylabel' ? '&nbsp;' : label || 'empty'}}</slot>
        </span>
        <i v-if="showClear && modelValue != null" class="p-dropdown-clear-icon pi pi-times" @click="onClearClick" v-bind="clearIconProps"></i>
        <div class="p-dropdown-trigger">
            <slot name="indicator">
                <span :class="dropdownIconClass" aria-hidden="true"></span>
            </slot>
        </div>
        <Portal :appendTo="appendTo">
            <transition name="p-connected-overlay" @enter="onOverlayEnter" @after-enter="onOverlayAfterEnter" @leave="onOverlayLeave" @after-leave="onOverlayAfterLeave">
                <div v-if="overlayVisible" :ref="overlayRef" :style="panelStyle" :class="panelStyleClass" @click="onOverlayClick" @keydown="onOverlayKeyDown" v-bind="panelProps">
                    <span ref="firstHiddenFocusableElementOnOverlay" role="presentation" aria-hidden="true" class="p-hidden-accessible p-hidden-focusable" :tabindex="0" @focus="onFirstHiddenFocus"></span>
                    <slot name="header" :value="modelValue" :options="visibleOptions"></slot>
                    <div v-if="filter" class="p-dropdown-header">
                        <div class="p-dropdown-filter-container">
                            <input type="text" ref="filterInput" :value="filterValue" @vnode-updated="onFilterUpdated" class="p-dropdown-filter p-inputtext p-component" :placeholder="filterPlaceholder"
                                role="searchbox" autocomplete="off" :aria-owns="id + '_list'" :aria-activedescendant="focusedOptionId"
                                @keydown="onFilterKeyDown" @blur="onFilterBlur" @input="onFilterChange" v-bind="filterInputProps"/>
                            <span class="p-dropdown-filter-icon pi pi-search"></span>
                        </div>
                        <span role="status" aria-live="polite" class="p-hidden-accessible">
                            {{filterResultMessageText}}
                        </span>
                    </div>
                    <div class="p-dropdown-items-wrapper" :style="{'max-height': virtualScrollerDisabled ? scrollHeight : ''}">
                        <VirtualScroller :ref="virtualScrollerRef" v-bind="virtualScrollerOptions" :items="visibleOptions" :style="{'height': scrollHeight}" :tabindex="-1" :disabled="virtualScrollerDisabled">
                            <template v-slot:content="{ styleClass, contentRef, items, getItemOptions, contentStyle, itemSize }">
                                <ul :ref="(el) => listRef(el, contentRef)" :id="id + '_list'" :class="['p-dropdown-items', styleClass]" :style="contentStyle" role="listbox">
                                    <template v-for="(option, i) of items" :key="getOptionRenderKey(option, getOptionIndex(i, getItemOptions))">
                                        <li v-if="isOptionGroup(option)" :id="id + '_' + getOptionIndex(i, getItemOptions)" :style="{height: itemSize ? itemSize + 'px' : undefined}" class="p-dropdown-item-group" role="option">
                                            <slot name="optiongroup" :option="option.optionGroup" :index="getOptionIndex(i, getItemOptions)">{{getOptionGroupLabel(option.optionGroup)}}</slot>
                                        </li>
                                        <li v-else v-ripple :id="id + '_' + getOptionIndex(i, getItemOptions)" :style="{height: itemSize ? itemSize + 'px' : undefined}"
                                            :class="['p-dropdown-item', {'p-highlight': isSelected(option), 'p-focus': focusedOptionIndex === getOptionIndex(i, getItemOptions), 'p-disabled': isOptionDisabled(option)}]"
                                            role="option" :aria-label="getOptionLabel(option)" :aria-selected="isSelected(option)" :aria-disabled="isOptionDisabled(option)" :aria-setsize="ariaSetSize" :aria-posinset="getAriaPosInset(getOptionIndex(i, getItemOptions))"
                                            @click="onOptionSelect($event, option)" @mousemove="onOptionMouseMove($event, getOptionIndex(i, getItemOptions))">
                                            <slot name="option" :option="option" :index="getOptionIndex(i, getItemOptions)">{{getOptionLabel(option)}}</slot>
                                        </li>
                                    </template>
                                    <li v-if="filterValue && (!items || (items && items.length === 0))" class="p-dropdown-empty-message" role="option">
                                        <slot name="emptyfilter">{{emptyFilterMessageText}}</slot>
                                    </li>
                                    <li v-else-if="(!options || (options && options.length === 0))" class="p-dropdown-empty-message" role="option">
                                        <slot name="empty">{{emptyMessageText}}</slot>
                                    </li>
                                </ul>
                                <span v-if="(!options || (options && options.length === 0))" role="status" aria-live="polite" class="p-hidden-accessible">
                                    {{emptyMessageText}}
                                </span>
                                <span role="status" aria-live="polite" class="p-hidden-accessible">
                                    {{selectedMessageText}}
                                </span>
                            </template>
                            <template v-if="$slots.loader" v-slot:loader="{ options }">
                                <slot name="loader" :options="options"></slot>
                            </template>
                        </VirtualScroller>
                    </div>
                    <slot name="footer" :value="modelValue" :options="visibleOptions"></slot>
                    <span ref="lastHiddenFocusableElementOnOverlay" role="presentation" aria-hidden="true" class="p-hidden-accessible p-hidden-focusable" :tabindex="0" @focus="onLastHiddenFocus"></span>
                </div>
            </transition>
        </Portal>
    </div>
</template>

<script>
import {ConnectedOverlayScrollHandler,ObjectUtils,DomHandler,ZIndexUtils,UniqueComponentId} from 'primevue/utils';
import OverlayEventBus from 'primevue/overlayeventbus';
import {FilterService} from 'primevue/api';
import Ripple from 'primevue/ripple';
import VirtualScroller from 'primevue/virtualscroller';
import Portal from 'primevue/portal';

export default {
    name: 'Dropdown',
    emits: ['update:modelValue', 'change', 'focus', 'blur', 'before-show', 'before-hide', 'show', 'hide', 'filter'],
    props: {
        modelValue: null,
        options: Array,
        optionLabel: null,
        optionValue: null,
        optionDisabled: null,
        optionGroupLabel: null,
        optionGroupChildren: null,
		scrollHeight: {
			type: String,
			default: '200px'
		},
		filter: Boolean,
        filterPlaceholder: String,
        filterLocale: String,
        filterMatchMode: {
            type: String,
            default: 'contains'
        },
        filterFields: {
            type: Array,
            default: null
        },
		editable: Boolean,
		placeholder: String,
		disabled: Boolean,
        dataKey: null,
        showClear: Boolean,
        inputId: String,
        inputStyle: null,
        inputClass: null,
        inputProps: null,
        panelStyle: null,
        panelClass: null,
        panelProps: null,
        filterInputProps: null,
        clearIconProps: null,
        appendTo: {
            type: String,
            default: 'body'
        },
        loading: {
            type: Boolean,
            default: false
        },
        loadingIcon: {
            type: String,
            default: 'pi pi-spinner pi-spin'
        },
        virtualScrollerOptions: {
            type: Object,
            default: null
        },
        autoOptionFocus: {
            type: Boolean,
            default: true
        },
        filterMessage: {
            type: String,
            default: null
        },
        selectionMessage: {
            type: String,
            default: null
        },
        emptySelectionMessage: {
            type: String,
            default: null
        },
        emptyFilterMessage: {
            type: String,
            default: null
        },
        emptyMessage: {
            type: String,
            default: null
        },
        tabindex: {
            type: Number,
            default: 0
        },
        'aria-label': {
            type: String,
            default: null
        },
        'aria-labelledby': {
            type: String,
            default: null
        }
    },
    outsideClickListener: null,
    scrollHandler: null,
    resizeListener: null,
    overlay: null,
    list: null,
    virtualScroller: null,
    searchTimeout: null,
    searchValue: null,
    isModelValueChanged: false,
    selectOnFocus: false,
    focusOnHover: false,
    data() {
        return {
            id: UniqueComponentId(),
            focused: false,
            focusedOptionIndex: -1,
            filterValue: null,
            overlayVisible: false
        }
    },
    watch: {
        modelValue() {
            this.isModelValueChanged = true;
        },
        options() {
            this.autoUpdateModel();
        }
    },
    mounted() {
        this.id = this.$attrs.id || this.id;

        this.autoUpdateModel();
    },
    updated() {
        if (this.overlayVisible && this.isModelValueChanged) {
            this.scrollInView(this.findSelectedOptionIndex());
        }

        this.isModelValueChanged = false;
    },
    beforeUnmount() {
        this.unbindOutsideClickListener();
        this.unbindResizeListener();

        if (this.scrollHandler) {
            this.scrollHandler.destroy();
            this.scrollHandler = null;
        }

        if (this.overlay) {
            ZIndexUtils.clear(this.overlay);
            this.overlay = null;
        }
    },
    methods: {
        getOptionIndex(index, fn) {
            return this.virtualScrollerDisabled ? index : (fn && fn(index)['index']);
        },
        getOptionLabel(option) {
            return this.optionLabel ? ObjectUtils.resolveFieldData(option, this.optionLabel) : option;
        },
        getOptionValue(option) {
            return this.optionValue ? ObjectUtils.resolveFieldData(option, this.optionValue) : option;
        },
        getOptionRenderKey(option, index) {
            return (this.dataKey ? ObjectUtils.resolveFieldData(option, this.dataKey) : this.getOptionLabel(option)) + '_' + index;
        },
        isOptionDisabled(option) {
            return this.optionDisabled ? ObjectUtils.resolveFieldData(option, this.optionDisabled) : false;
        },
        isOptionGroup(option) {
            return this.optionGroupLabel && option.optionGroup && option.group;
        },
        getOptionGroupLabel(optionGroup) {
            return ObjectUtils.resolveFieldData(optionGroup, this.optionGroupLabel);
        },
        getOptionGroupChildren(optionGroup) {
            return ObjectUtils.resolveFieldData(optionGroup, this.optionGroupChildren);
        },
        getAriaPosInset(index) {
            return (this.optionGroupLabel ? index - this.visibleOptions.slice(0, index).filter(option => this.isOptionGroup(option)).length : index) + 1;
        },
        show(isFocus) {
            this.$emit('before-show');
            this.overlayVisible = true;
            this.focusedOptionIndex = this.focusedOptionIndex !== -1 ? this.focusedOptionIndex : (this.autoOptionFocus ? this.findFirstFocusedOptionIndex() : -1);

            isFocus && this.$refs.focusInput.focus();
        },
        hide(isFocus) {
            const _hide = () => {
                this.$emit('before-hide');
                this.overlayVisible = false;
                this.focusedOptionIndex = -1;
                this.searchValue = '';

                isFocus && this.$refs.focusInput.focus();
            }

            setTimeout(() => { _hide() }, 0); // For ScreenReaders
        },
        onFocus(event) {
            this.focused = true;
            this.focusedOptionIndex = this.overlayVisible && this.autoOptionFocus ? this.findFirstFocusedOptionIndex() : -1;
            this.overlayVisible && this.scrollInView(this.focusedOptionIndex);
            this.$emit('focus', event);
        },
        onBlur(event) {
            this.focused = false;
            this.focusedOptionIndex = -1;
            this.searchValue = '';
            this.$emit('blur', event);
        },
        onKeyDown(event) {
            switch (event.code) {
                case 'ArrowDown':
                    this.onArrowDownKey(event);
                    break;

                case 'ArrowUp':
                    this.onArrowUpKey(event, this.editable);
                    break;

                case 'ArrowLeft':
                case 'ArrowRight':
                    this.onArrowLeftKey(event, this.editable);
                    break;

                case 'Home':
                    this.onHomeKey(event, this.editable);
                    break;

                case 'End':
                    this.onEndKey(event, this.editable);
                    break;

                case 'PageDown':
                    this.onPageDownKey(event);
                    break;

                case 'PageUp':
                    this.onPageUpKey(event);
                    break;

                case 'Space':
                    this.onSpaceKey(event, this.editable);
                    break;

                case 'Enter':
                    this.onEnterKey(event);
                    break;

                case 'Escape':
                    this.onEscapeKey(event);
                    break;

                case 'Tab':
                    this.onTabKey(event);
                    break;

                case 'Backspace':
                    this.onBackspaceKey(event, this.editable);
                    break;

                case 'ShiftLeft':
                case 'ShiftRight':
                    //NOOP
                    break;

                default:
                    if (ObjectUtils.isPrintableCharacter(event.key)) {
                        !this.overlayVisible && this.show();
                        !this.editable && this.searchOptions(event, event.key);
                    }

                    break;
            }
        },
        onEditableInput(event) {
            const value = event.target.value;

            this.searchValue = '';
            const matched = this.searchOptions(event, value);
            !matched && (this.focusedOptionIndex = -1);

            this.$emit('update:modelValue', value);
        },
        onContainerClick(event) {
            if (this.disabled || this.loading) {
                return;
            }

            if (DomHandler.hasClass(event.target, 'p-dropdown-clear-icon') || event.target.tagName === 'INPUT') {
                return;
            }
            else if (!this.overlay || !this.overlay.contains(event.target)) {
                this.overlayVisible ? this.hide(true) : this.show(true);
            }
        },
        onClearClick(event) {
            this.updateModel(event, null);
        },
        onFirstHiddenFocus(event) {
            const relatedTarget = event.relatedTarget;

            if (relatedTarget === this.$refs.focusInput) {
                const firstFocusableEl = DomHandler.getFirstFocusableElement(this.overlay, ':not(.p-hidden-focusable)');
                firstFocusableEl && firstFocusableEl.focus();
            }
            else {
                this.$refs.focusInput.focus();
            }
        },
        onLastHiddenFocus() {
            this.$refs.firstHiddenFocusableElementOnOverlay.focus();
        },
        onOptionSelect(event, option) {
            const value = this.getOptionValue(option);

            this.updateModel(event, value);
            this.hide(true);
        },
        onOptionMouseMove(event, index) {
            if (this.focusOnHover) {
                this.changeFocusedOptionIndex(event, index);
            }
        },
        onFilterChange(event) {
            const value = event.target.value;

            this.filterValue = value;
            this.focusedOptionIndex = -1;
            this.$emit('filter', { originalEvent: event, value });

            !this.virtualScrollerDisabled && this.virtualScroller.scrollToIndex(0);
        },
        onFilterKeyDown(event) {
            switch (event.code) {
                case 'ArrowDown':
                    this.onArrowDownKey(event);
                    break;

                case 'ArrowUp':
                    this.onArrowUpKey(event, true);
                    break;

                case 'ArrowLeft':
                case 'ArrowRight':
                    this.onArrowLeftKey(event, true);
                    break;

                case 'Home':
                    this.onHomeKey(event, true);
                    break;

                case 'End':
                    this.onEndKey(event, true);
                    break;

                case 'Enter':
                    this.onEnterKey(event);
                    break;

                case 'Escape':
                    this.onEscapeKey(event);
                    break;

                case 'Tab':
                    this.onTabKey(event, true);
                    break;

                default:
                    break;
            }
        },
        onFilterBlur() {
            this.focusedOptionIndex = -1;
        },
        onFilterUpdated() {
            if (this.overlayVisible) {
                this.alignOverlay();
            }
        },
        onOverlayClick(event) {
            OverlayEventBus.emit('overlay-click', {
                originalEvent: event,
                target: this.$el
            });
        },
        onOverlayKeyDown(event) {
            switch (event.code) {
                case 'Escape':
                    this.onEscapeKey(event);
                    break;

                default:
                    break;
            }
        },
        onArrowDownKey(event) {
            const optionIndex = this.focusedOptionIndex !== -1 ? this.findNextOptionIndex(this.focusedOptionIndex) : this.findFirstFocusedOptionIndex();

            this.changeFocusedOptionIndex(event, optionIndex);

            !this.overlayVisible && this.show();
            event.preventDefault();
        },
        onArrowUpKey(event, pressedInInputText = false) {
            if (event.altKey && !pressedInInputText) {
                if (this.focusedOptionIndex !== -1) {
                    this.onOptionSelect(event, this.visibleOptions[this.focusedOptionIndex]);
                }

                this.overlayVisible && this.hide();
                event.preventDefault();
            }
            else {
                const optionIndex = this.focusedOptionIndex !== -1 ? this.findPrevOptionIndex(this.focusedOptionIndex) : this.findLastFocusedOptionIndex();

                this.changeFocusedOptionIndex(event, optionIndex);

                !this.overlayVisible && this.show();
                event.preventDefault();
            }
        },
        onArrowLeftKey(event, pressedInInputText = false) {
            pressedInInputText && (this.focusedOptionIndex = -1);
        },
        onHomeKey(event, pressedInInputText = false) {
            if (pressedInInputText) {
                event.currentTarget.setSelectionRange(0, 0);
                this.focusedOptionIndex = -1;
            }
            else {
                this.changeFocusedOptionIndex(event, this.findFirstOptionIndex());

                !this.overlayVisible && this.show();
            }

            event.preventDefault();
        },
        onEndKey(event, pressedInInputText = false) {
            if (pressedInInputText) {
                const target = event.currentTarget;
                const len = target.value.length;
                target.setSelectionRange(len, len);
                this.focusedOptionIndex = -1;
            }
            else {
                this.changeFocusedOptionIndex(event, this.findLastOptionIndex());

                !this.overlayVisible && this.show();
            }

            event.preventDefault();
        },
        onPageUpKey(event) {
            this.scrollInView(0);
            event.preventDefault();
        },
        onPageDownKey(event) {
            this.scrollInView(this.visibleOptions.length - 1);
            event.preventDefault();
        },
        onEnterKey(event) {
            if (!this.overlayVisible) {
                this.onArrowDownKey(event);
            }
            else {
                if (this.focusedOptionIndex !== -1) {
                    this.onOptionSelect(event, this.visibleOptions[this.focusedOptionIndex]);
                }

                this.hide();
            }

            event.preventDefault();
        },
        onSpaceKey(event, pressedInInputText = false) {
            !pressedInInputText && this.onEnterKey(event);
        },
        onEscapeKey(event) {
            this.overlayVisible && this.hide(true);
            event.preventDefault();
        },
        onTabKey(event, pressedInInputText = false) {
            if (!pressedInInputText) {
                if (this.overlayVisible && this.hasFocusableElements()) {
                    this.$refs.firstHiddenFocusableElementOnOverlay.focus();

                    event.preventDefault();
                }
                else {
                    if (this.focusedOptionIndex !== -1) {
                        this.onOptionSelect(event, this.visibleOptions[this.focusedOptionIndex]);
                    }

                    this.overlayVisible && this.hide(this.filter);
                }
            }
        },
        onBackspaceKey(event, pressedInInputText = false) {
            if (pressedInInputText) {
                !this.overlayVisible && this.show();
            }
        },
        onOverlayEnter(el) {
            ZIndexUtils.set('overlay', el, this.$primevue.config.zIndex.overlay);
            this.alignOverlay();
            this.scrollInView();
        },
        onOverlayAfterEnter() {
            this.bindOutsideClickListener();
            this.bindScrollListener();
            this.bindResizeListener();

            this.$emit('show');
        },
        onOverlayLeave() {
            this.unbindOutsideClickListener();
            this.unbindScrollListener();
            this.unbindResizeListener();

            this.$emit('hide');
            this.overlay = null;
        },
        onOverlayAfterLeave(el) {
            ZIndexUtils.clear(el);
        },
        alignOverlay() {
            if (this.appendTo === 'self') {
                DomHandler.relativePosition(this.overlay, this.$el);
            }
            else {
                this.overlay.style.minWidth = DomHandler.getOuterWidth(this.$el) + 'px';
                DomHandler.absolutePosition(this.overlay, this.$el);
            }
        },
        bindOutsideClickListener() {
            if (!this.outsideClickListener) {
                this.outsideClickListener = (event) => {
                    if (this.overlayVisible && this.overlay && !this.$el.contains(event.target) && !this.overlay.contains(event.target)) {
                        this.hide();
                    }
                };
                document.addEventListener('click', this.outsideClickListener);
            }
        },
        unbindOutsideClickListener() {
            if (this.outsideClickListener) {
                document.removeEventListener('click', this.outsideClickListener);
                this.outsideClickListener = null;
            }
        },
        bindScrollListener() {
            if (!this.scrollHandler) {
                this.scrollHandler = new ConnectedOverlayScrollHandler(this.$refs.container, () => {
                    if (this.overlayVisible) {
                        this.hide();
                    }
                });
            }

            this.scrollHandler.bindScrollListener();
        },
        unbindScrollListener() {
            if (this.scrollHandler) {
                this.scrollHandler.unbindScrollListener();
            }
        },
        bindResizeListener() {
            if (!this.resizeListener) {
                this.resizeListener = () => {
                    if (this.overlayVisible && !DomHandler.isTouchDevice()) {
                        this.hide();
                    }
                };
                window.addEventListener('resize', this.resizeListener);
            }
        },
        unbindResizeListener() {
            if (this.resizeListener) {
                window.removeEventListener('resize', this.resizeListener);
                this.resizeListener = null;
            }
        },
        hasFocusableElements() {
            return DomHandler.getFocusableElements(this.overlay, ':not(.p-hidden-focusable)').length > 0;
        },
        isOptionMatched(option) {
            return this.isValidOption(option) && this.getOptionLabel(option).toLocaleLowerCase(this.filterLocale).startsWith(this.searchValue.toLocaleLowerCase(this.filterLocale));
        },
        isValidOption(option) {
            return option && !(this.isOptionDisabled(option) || this.isOptionGroup(option));
        },
        isValidSelectedOption(option) {
            return this.isValidOption(option) && this.isSelected(option);
        },
        isSelected(option) {
            return ObjectUtils.equals(this.modelValue, this.getOptionValue(option), this.equalityKey);
        },
        findFirstOptionIndex() {
            return this.visibleOptions.findIndex(option => this.isValidOption(option));
        },
        findLastOptionIndex() {
            return ObjectUtils.findLastIndex(this.visibleOptions, option => this.isValidOption(option));
        },
        findNextOptionIndex(index) {
            const matchedOptionIndex = index < (this.visibleOptions.length - 1) ? this.visibleOptions.slice(index + 1).findIndex(option => this.isValidOption(option)) : -1;
            return matchedOptionIndex > -1 ? matchedOptionIndex + index + 1 : index;
        },
        findPrevOptionIndex(index) {
            const matchedOptionIndex = index > 0 ? ObjectUtils.findLastIndex(this.visibleOptions.slice(0, index), option => this.isValidOption(option)) : -1;
            return matchedOptionIndex > -1 ? matchedOptionIndex : index;
        },
        findSelectedOptionIndex() {
            return this.hasSelectedOption ? this.visibleOptions.findIndex(option => this.isValidSelectedOption(option)) : -1;
        },
        findFirstFocusedOptionIndex() {
            const selectedIndex = this.findSelectedOptionIndex();
            return selectedIndex < 0 ? this.findFirstOptionIndex() : selectedIndex;
        },
        findLastFocusedOptionIndex() {
            const selectedIndex = this.findSelectedOptionIndex();
            return selectedIndex < 0 ? this.findLastOptionIndex() : selectedIndex;
        },
        searchOptions(event, char) {
            this.searchValue = (this.searchValue || '') + char;

            let optionIndex = -1;
            let matched = false;

            if (this.focusedOptionIndex !== -1) {
                optionIndex = this.visibleOptions.slice(this.focusedOptionIndex).findIndex(option => this.isOptionMatched(option));
                optionIndex = optionIndex === -1 ? this.visibleOptions.slice(0, this.focusedOptionIndex).findIndex(option => this.isOptionMatched(option)) : optionIndex + this.focusedOptionIndex;
            }
            else {
                optionIndex = this.visibleOptions.findIndex(option => this.isOptionMatched(option));
            }

            if (optionIndex !== -1) {
                matched = true;
            }

            if (optionIndex === -1 && this.focusedOptionIndex === -1) {
                optionIndex = this.findFirstFocusedOptionIndex();
            }

            if (optionIndex !== -1) {
                this.changeFocusedOptionIndex(event, optionIndex);
            }

            if (this.searchTimeout) {
                clearTimeout(this.searchTimeout);
            }

            this.searchTimeout = setTimeout(() => {
                this.searchValue = '';
                this.searchTimeout = null;
            }, 500);

            return matched;
        },
        changeFocusedOptionIndex(event, index) {
            if (this.focusedOptionIndex !== index) {
                this.focusedOptionIndex = index;
                this.scrollInView();

                if (this.selectOnFocus) {
                    this.updateModel(event, this.getOptionValue(this.visibleOptions[index]));
                }
            }
        },
        scrollInView(index = -1) {
            const id = index !== -1 ? `${this.id}_${index}` : this.focusedOptionId;
            const element = DomHandler.findSingle(this.list, `li[id="${id}"]`);
            if (element) {
                element.scrollIntoView && element.scrollIntoView({ block: 'nearest', inline: 'start' });
            }
            else if (!this.virtualScrollerDisabled) {
                setTimeout(() => {
                    this.virtualScroller && this.virtualScroller.scrollToIndex(index !== -1 ? index : this.focusedOptionIndex);
                }, 0);
            }
        },
        autoUpdateModel() {
            if (this.selectOnFocus && this.autoOptionFocus && !this.hasSelectedOption) {
                this.focusedOptionIndex = this.findFirstFocusedOptionIndex();
                const value = this.getOptionValue(this.visibleOptions[this.focusedOptionIndex]);
                this.updateModel(null, value);
            }
        },
        updateModel(event, value) {
            this.$emit('update:modelValue', value);
            this.$emit('change', { originalEvent: event, value });
        },
        flatOptions(options) {
            return (options || []).reduce((result, option, index) => {
                result.push({ optionGroup: option, group: true, index });

                const optionGroupChildren = this.getOptionGroupChildren(option);
                optionGroupChildren && optionGroupChildren.forEach(o => result.push(o));

                return result;
            }, []);
        },
        overlayRef(el) {
            this.overlay = el;
        },
        listRef(el, contentRef) {
            this.list = el;
            contentRef && contentRef(el); // For VirtualScroller
        },
        virtualScrollerRef(el) {
            this.virtualScroller = el;
        }
    },
    computed: {
        containerClass() {
            return ['p-dropdown p-component p-inputwrapper', {
                'p-disabled': this.disabled,
                'p-dropdown-clearable': this.showClear && !this.disabled,
                'p-focus': this.focused,
                'p-inputwrapper-filled': this.modelValue,
                'p-inputwrapper-focus': this.focused || this.overlayVisible,
                'p-overlay-open': this.overlayVisible
            }];
        },
        inputStyleClass() {
            return ['p-dropdown-label p-inputtext', this.inputClass, {
                'p-placeholder': !this.editable && this.label === this.placeholder,
                'p-dropdown-label-empty': !this.editable && !this.$slots['value'] && (this.label === 'p-emptylabel' || this.label.length === 0)
            }];
        },
        panelStyleClass() {
            return ['p-dropdown-panel p-component', this.panelClass, {
                'p-input-filled': this.$primevue.config.inputStyle === 'filled',
                'p-ripple-disabled': this.$primevue.config.ripple === false
            }];
        },
        dropdownIconClass() {
            return ['p-dropdown-trigger-icon', (this.loading ? this.loadingIcon : 'pi pi-chevron-down')];
        },
        visibleOptions() {
            const options = this.optionGroupLabel ? this.flatOptions(this.options) : (this.options || []);

            return this.filterValue ? FilterService.filter(options, this.searchFields, this.filterValue, this.filterMatchMode, this.filterLocale) : options;
        },
        hasSelectedOption() {
            return ObjectUtils.isNotEmpty(this.modelValue);
        },
        label() {
            const selectedOptionIndex = this.findSelectedOptionIndex();
            return selectedOptionIndex !== -1 ? this.getOptionLabel(this.visibleOptions[selectedOptionIndex]) : (this.placeholder || 'p-emptylabel');
        },
        editableInputValue() {
            const selectedOptionIndex = this.findSelectedOptionIndex();
            return selectedOptionIndex !== -1 ? this.getOptionLabel(this.visibleOptions[selectedOptionIndex]) : (this.modelValue || '');
        },
        equalityKey() {
            return this.optionValue ? null : this.dataKey;
        },
        searchFields() {
            return this.filterFields || [this.optionLabel];
        },
        filterResultMessageText() {
            return ObjectUtils.isNotEmpty(this.visibleOptions) ? this.filterMessageText.replaceAll('{0}', this.visibleOptions.length) : this.emptyFilterMessageText;
        },
        filterMessageText() {
            return this.filterMessage || this.$primevue.config.locale.searchMessage || '';
        },
        emptyFilterMessageText() {
            return this.emptyFilterMessage || this.$primevue.config.locale.emptySearchMessage || this.$primevue.config.locale.emptyFilterMessage || '';
        },
        emptyMessageText() {
            return this.emptyMessage || this.$primevue.config.locale.emptyMessage || '';
        },
        selectionMessageText() {
            return this.selectionMessage || this.$primevue.config.locale.selectionMessage || '';
        },
        emptySelectionMessageText() {
            return this.emptySelectionMessage || this.$primevue.config.locale.emptySelectionMessage || '';
        },
        selectedMessageText() {
            return this.hasSelectedOption ? this.selectionMessageText.replaceAll('{0}', '1') : this.emptySelectionMessageText;
        },
        focusedOptionId() {
            return this.focusedOptionIndex !== -1 ? `${this.id}_${this.focusedOptionIndex}` : null;
        },
        ariaSetSize() {
            return this.visibleOptions.filter(option => !this.isOptionGroup(option)).length;
        },
        virtualScrollerDisabled() {
            return !this.virtualScrollerOptions;
        }
    },
    directives: {
        'ripple': Ripple
    },
    components: {
        'VirtualScroller': VirtualScroller,
        'Portal': Portal
    }
}
</script>

<style>
.p-dropdown {
    display: inline-flex;
    cursor: pointer;
    position: relative;
    user-select: none;
}

.p-dropdown-clear-icon {
    position: absolute;
    top: 50%;
    margin-top: -.5rem;
}

.p-dropdown-trigger {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
}

.p-dropdown-label {
    display: block;
    white-space: nowrap;
    overflow: hidden;
    flex: 1 1 auto;
    width: 1%;
    text-overflow: ellipsis;
    cursor: pointer;
}

.p-dropdown-label-empty {
    overflow: hidden;
    opacity: 0;
}

input.p-dropdown-label  {
    cursor: default;
}

.p-dropdown .p-dropdown-panel {
    min-width: 100%;
}

.p-dropdown-panel {
    position: absolute;
    top: 0;
    left: 0;
}

.p-dropdown-items-wrapper {
    overflow: auto;
}

.p-dropdown-item {
    cursor: pointer;
    font-weight: normal;
    white-space: nowrap;
    position: relative;
    overflow: hidden;
}

.p-dropdown-item-group {
    cursor: auto;
}

.p-dropdown-items {
    margin: 0;
    padding: 0;
    list-style-type: none;
}

.p-dropdown-filter {
    width: 100%;
}

.p-dropdown-filter-container {
    position: relative;
}

.p-dropdown-filter-icon {
    position: absolute;
    top: 50%;
    margin-top: -.5rem;
}

.p-fluid .p-dropdown {
    display: flex;
}

.p-fluid .p-dropdown .p-dropdown-label {
    width: 1%;
}
</style>
