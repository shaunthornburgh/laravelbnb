<template>
    <div class="sm:pl-3 sm:order-none order-first mb-7 sm:mb-0">
        <div class="mb-3 font-semibold" style="color: gray">
            Check Availability
            <span v-if="noAvailability" class="text-red-500"> - Not Available</span>
            <span v-if="hasAvailability" class="text-green-500"> - Available</span>
        </div>
        <form action="#" method="get">
            <div class="flex mb-4">
                <div class="flex flex-col w-6/12 pr-1.5">
                    <label
                        class="uppercase font-semibold text-xs mb-2"
                        style="color: gray"
                        for="date-from"
                    >from</label>
                    <input
                        class="border rounded pl-2 pt-1 pb-1"
                        id="date-from"
                        type="date"
                        name="date-from"
                        v-model="from"
                        @keyup.enter="check"
                        :class="[{'border-red-500': errorFor('from')}]"
                    />
                    <validation-errors :errors="errorFor('from')"></validation-errors>
                </div>
                <div class="flex flex-col w-6/12 pl-1.5">
                    <label
                        class="uppercase font-semibold text-xs mb-2"
                        style="color: gray"
                        for="date-to"
                    >to</label>
                    <input
                        class="border rounded pl-2 pt-1 pb-1"
                        id="date-to"
                        type="date"
                        name="date-to"
                        v-model="to"
                        @keyup.enter="check"
                        :class="[{'border-red-500': errorFor('to')}]"
                    />
                    <validation-errors :errors="errorFor('to')"></validation-errors>
                </div>
            </div>
            <button
                type="submit"
                @click="check"
                :disabled="loading"
                class="inline-flex items-center justify-center px-4 py-2 border border-transparent text-base font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 w-full"
            >Check availability</button>
        </form>
    </div>
</template>

<script>
    import { is422 } from "../shared/utils/response";
    import validationErrors from "./../shared/mixins/validationErrors";

    export default {
        mixins: [validationErrors],
        props: {
            bookableId: [String, Number]
        },
        data() {
            return {
                from: this.$store.state.lastSearch.from,
                to: this.$store.state.lastSearch.to,
                loading: false,
                status: null,
            }
        },
        methods: {
            async check() {
                this.loading = true;
                this.errors = null;

                this.$store.dispatch("setLastSearch", {
                    from: this.from,
                    to: this.to
                });

                try {
                    this.status = (await axios.get(
                        `/api/bookables/${this.bookableId}/availability?from=${this.from}&to=${this.to}`
                    )).status;
                    this.$emit("availability", this.hasAvailability);
                } catch (err) {
                    if (is422(err)) {
                        this.errors = err.response.data.errors;
                    }

                    this.status = err.response.status;
                    this.$emit("availability", this.hasAvailability);
                }

                this.loading = false;
            }
        },
        computed: {
            hasErrors() {
                return 422 === this.status && this.errors !== null;
            },
            hasAvailability() {
                return 200 === this.status;
            },
            noAvailability() {
                return 404 === this.status;
            }
        }
    };
</script>
