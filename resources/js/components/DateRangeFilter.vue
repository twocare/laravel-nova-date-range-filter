<template>
    <div class="date-range-filter">
        <h3 class="text-sm uppercase tracking-wide text-80 bg-30 p-3">{{ filter.name }}</h3>

        <div class="p-2">
            <div class="input-wrapper">
                <date-range-picker
                    ref="dateRangePickerComponent"
                    class="w-full form-control form-control-sm form-input form-input-bordered"
                    dusk="date-range-filter"
                    name="date-range-filter"
                    autocomplete="off"
                    :disabled="disabled"
                    :placeholder="placeholder"
                    :value="value"
                    :allow-input="allowInput"
                    :date-format="dateFormat"
                    :enable-time="enableTime"
                    :enable-seconds="enableSeconds"
                    :locale="locale"
                    :max-date="maxDate"
                    :min-date="minDate"
                    :shorthand-current-month="shorthandCurrentMonth"
                    :show-months="showMonths"
                    :time24hr="time24hr"
                    :week-numbers="weekNumbers"
                    :first-day-of-week="firstDayOfWeek"
                    @input.prevent=""
                    @change="handleChange"
                    @ready="setDisplayValue"
                />

                <button
                    class="reset-button btn btn-sm fa fa-times"
                    :class="{ 'hidden': !this.filter.currentValue }"
                    @click="resetFilter"
                ></button>
            </div>
        </div>
    </div>
</template>

<script>
    import flatpickr from 'flatpickr';

    export default {
        props: {
            resourceName: {
                type: String,
                required: true,
            },
            filterKey: {
                type: String,
                required: true,
            },
        },

        data: function () {
            return {
                displayValue: '',
            };
        },

        methods: {
            handleChange: function (value) {
                this.$store.commit(`${this.resourceName}/updateFilterState`, {
                    filterClass: this.filterKey,
                    value: value.length ? value : '',
                });

                this.$emit('change');
            },

            setDisplayValue: function () {
                const flatpickrConfig = this.$refs.dateRangePickerComponent.getFlatpickrConfig();
                const rangeSeparator = flatpickrConfig.locale.rangeSeparator || 'to';

                if (!Array.isArray(this.filter.currentValue) || this.filter.currentValue.length !== 2) {
                    return;
                }

                this.displayValue = this.filter.currentValue
                    .map((value) => flatpickr.formatDate(flatpickr.parseDate(value), this.dateFormat))
                    .join(` ${rangeSeparator} `);
            },

            resetFilter: function () {
                this.$refs.dateRangePickerComponent.clear();
            },
        },

        computed: {
            filter: function () {
                return this.$store.getters[`${this.resourceName}/getFilter`](this.filterKey);
            },

            disabled: function () {
                return this.filter.disabled !== undefined ? this.filter.disabled : false;
            },

            placeholder: function () {
                return this.filter.placeholder || this.__('Choose date range');
            },

            value: function () {
                return this.displayValue;
            },

            allowInput: function () {
                return this.filter.allowInput !== undefined ? this.filter.allowInput : true;
            },

            dateFormat: function () {
                return this.filter.dateFormat || 'Y-m-d';
            },

            enableTime: function () {
                return this.filter.enableTime !== undefined ? this.filter.enableTime : false;
            },

            enableSeconds: function () {
                return this.filter.enableSeconds !== undefined ? this.filter.enableSeconds : false;
            },

            locale: function () {
                if (!this.filter.locale) {
                    return 'default';
                }

                const locale = this.filter.locale.split('-')[0];

                // There is no english language file.
                if (locale === 'en') {
                    return 'default';
                }

                return require(`flatpickr/dist/l10n/${locale}.js`).default[locale];
            },

            maxDate: function () {
                return this.filter.maxDate || null;
            },

            minDate: function () {
                return this.filter.minDate || null;
            },

            shorthandCurrentMonth: function () {
                return this.filter.shorthandCurrentMonth !== undefined ? this.filter.shorthandCurrentMonth : false;
            },

            showMonths: function () {
                return this.filter.showMonths || 1;
            },

            time24hr: function () {
                return this.filter.time24hr !== undefined ? this.filter.time24hr : true;
            },

            weekNumbers: function () {
                return this.filter.weekNumbers !== undefined ? this.filter.weekNumbers : false;
            },

            firstDayOfWeek: function () {
                return this.filter.firstDayOfWeek || 0;
            },
        },
    };
</script>