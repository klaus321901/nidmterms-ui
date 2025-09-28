<template>
    <div>
        <div class="panel panel-default">
            <div class="panel-body scroll">
                <vue-form-generator
                        :key="selectedTerm"
                        :schema="schema"
                        :model="model"
                        :options="formOptions">
                </vue-form-generator>
            </div>
        </div>
        <b-button class="button" @click="onSave">Save</b-button>
    </div>
</template>

<style scoped>
    .scroll {
        height: 880px;
        overflow-y: auto;
    }

    h1 {
        text-align: center;
        font-size: 36px;
        margin-top: 20px;
        margin-bottom: 10px;
        font-weight: 500;
    }

    fieldset {
        border: 0;
    }

    .panel {
        margin-bottom: 20px;
        background-color: #fff;
        border: 1px solid #ddd;
        border-radius: 4px;
        -webkit-box-shadow: 0 1px 1px rgba(0, 0, 0, .05);
        box-shadow: 0 1px 1px rgba(0, 0, 0, .05);
    }

    .panel-body {
        padding: 15px;
        overflow-y: scroll;
    }

    .button { }

    .field-checklist .wrapper {
        width: 100%;
    }

    .fade-enter-active, .fade-leave-active {
        transition: opacity .5s;
    }

    .fade-enter, .fade-leave-to {
        opacity: 0;
    }

    .group-one-class {
        background-color: red;
        width: 100%;
    }

    .checklist-class {
        width: 100%;
        box-sizing: border-box;
        display: block;
    }
</style>

<script>
    import VueFormGenerator from 'vue-form-generator';
    import 'vue-form-generator/dist/vfg.css';
    import _ from 'lodash';

    export default {
        name: "TermProperties",
        props: {
            selectedTerm: String,
            init: Object,
            searchResults: Array,
            selectedConcepts: Array
        },
        components: {
            "vue-form-generator": VueFormGenerator.component
        },
        data() {
            return {
                model: {
                    label: '',
                    sourceVariable: this.selectedTerm,
                    description: '',
                    url: '',
                    valueType: '',
                    isAbout: [],
                    type: [],
                    datumType: '',
                    unitCode: '',
                    maxValue: '' ,
                    minValue: '',
                    allowableValues: [],
                    choicesName: [],
                    derivative: '',
                    citation: '',
                    measureOf: '',
                    provenance: '',
                    required: '',
                    choicesValue: []
                },
                schema: {
                    groups: [
                        {
                            styleClasses: "group-one-class",
                            fields: [
                                {
                                    type: "input",
                                    inputType: "text",
                                    label: "Label",
                                    model: "label",
                                    validator: VueFormGenerator.validators.string
                                },
                                {
                                    type: "input",
                                    inputType: "text",
                                    label: "Description",
                                    model: "description",
                                    validator: VueFormGenerator.validators.string
                                },
                                {
                                    type: "checklist",
                                    label: "isAbout",
                                    model: "isAbout",
                                    required: false,
                                    multiSelect: true,
                                    featured: true,
                                    styleClasses: 'checklist-class',
                                    values() {
                                        return [
                                            { name: "Option A", value: "A" },
                                            { name: "Option B", value: "B" },
                                            { name: "Option C", value: "C" }
                                        ];
                                    },
                                    hint: "An explanation of the nature, scope, or meaning of the new term.",
                                    help: "Check right column for definition of the related concept terms",
                                    validator: VueFormGenerator.validators.array,
                                },
                                {
                                    type: "select",
                                    inputType: "text",
                                    label: "valueType",
                                    model: "valueType",
                                    hint: "A value representation such as integer, float, string, date/time",
                                    featured: true,
                                    readonly: false,
                                    required: true,
                                    multiSelect: false,
                                    noneSelectedText: 'Select one',
                                    values: ['string', 'boolean', 'integer', 'float', 'double', 'duration', 'datetime', 'time', 'date', 'anyURI', 'complexType']
                                },
                                {
                                    type: "input",
                                    inputType: 'text',
                                    label: "datumType",
                                    model: "datumType",
                                    validator: VueFormGenerator.validators.string
                                },
                                {
                                    type: "input",
                                    inputType: 'text',
                                    label: "Unit",
                                    model: "unitCode",
                                    validator: VueFormGenerator.validators.string
                                },
                                {
                                    type: "input",
                                    inputType: "number",
                                    label: "Min Value",
                                    model: "minValue",
                                    validator: VueFormGenerator.validators.number
                                },
                                {
                                    type: "input",
                                    inputType: 'number',
                                    label: "Max Value",
                                    model: "maxValue",
                                    validator: VueFormGenerator.validators.number
                                }
                            ]
                        },
                        {
                            styleClasses: "choices-group",
                            legend: "Choices",
                            fields: [
                                {
                                    type: "array",
                                    label: "Name",
                                    model: "choicesName",
                                    featured: true,
                                    showRemoveButton: true,
                                    styleClasses: "col-sm-5",
                                },
                                {
                                    type: "array",
                                    label: "Value",
                                    model: "choicesValue",
                                    featured: true,
                                    showRemoveButton: true,
                                    styleClasses: "col-sm-5",
                                }
                            ]
                        }
                    ]
                },
                interlexTerms: [],
                formOptions: {
                    validateAfterLoad: true,
                    validateAfterChanged: true,
                    validateAsync: true
                },
                final_list: []
            };
        },
        methods: {
            onSave() {
                this.$emit('saveResponse', this.selectedTerm, this.model, this.final_list);
                document.body.scrollTop = 0;
                document.documentElement.scrollTop = 0;
            },
        },
        mounted() {
            if (this.init) {
                this.model = this.init;
            }
        },
    };
</script>
