<template>
    <div id="annotate">
        <div>
            <p class="upload"><b> Upload a CSV or TSV file to annotate:  </b></p>
            <input type="file" @change="onFileSelected">
            <!--            <button @click="onFileSelected">Upload</button>-->

        </div>
        <div class="download-btn" v-if="isAnnotated">
            <b-button variant="primary" @click="downloadJSONLD">Download JSON</b-button>
        </div>
        <b-container fluid class="col-container" v-if="sourceList.length">
            <div id="sidebar-left">
                <div class="col-header">
                    <p><b>Terms</b></p>
                </div>
                <div>
                    <SourceVariableList :terms1="sourceList" :completed="completed"
                                        :broadenConceptSearch="broadenConceptSearch"
                                        v-on:termSearchResult="searchList"
                                        v-on:termSelect="showTermProperties"
                                        v-on:nidmConceptFound="showBroaden"
                    ></SourceVariableList>
                </div>
            </div>
            <div id="main-content" v-if="selectedTerm">
                <div class="col-header">
                    <p><b>Term Properties</b></p>
                </div>
                <transition name="fade" mode="out-in">
                    <TermProperties :key="selectedTerm" :init="responses[selectedTerm]"
                                    v-if="showForm" :selectedTerm="selectedTerm"
                                    :searchResults="interlexTerms"
                                    :selectedConcepts="concepts[selectedTerm]"
                                    v-on:saveResponse="setResponse"></TermProperties>
                </transition>
            </div>
            <div id="sidebar-right" v-if="selectedTerm">
                <div class="col-header" style="display:flex">
                    <div class="header-text" style="width: 40%"><p><b>Concept Search</b></p></div>
                    <div class="concept-search">
                        <b-form-input size="sm" class="mr-sm-2 header-text" placeholder="Search" style="width: auto"
                                      v-model="initSearch"
                                      @keydown.enter.native="test_enter_handler"
                        ></b-form-input>
                    </div>
                </div>
                <InterlexSearchList :key="selectedTerm" :terms="interlexTerms" :broaden="showBroadenButton"
                                    :selectedTerm="selectedTerm"
                                    :init="concepts[selectedTerm]"
                                    v-on:broadenSearch="broadenSearchList"
                                    v-on:termSelect="updateConceptList"
                                    v-on:saveCheckedConcepts="setConcepts"
                ></InterlexSearchList>
            </div>
        </b-container>
    </div>
</template>

<script>
    import Papa from 'papaparse';
    import _ from 'lodash';
    import { saveAs } from 'file-saver';
    import 'vue-form-generator/dist/vfg.css';
    import SourceVariableList from './SourceVariableList';
    import InterlexSearchList from "./InterlexSearchList";
    import TermProperties from "./TermProperties";
    import axios from "axios";

    const API_KEY = 'CfufpoSNVXujv7h14HFHI4dL9p36mxCJ';

    export default {
        name: 'Annotate',
        components: {
            TermProperties,
            InterlexSearchList,
            SourceVariableList
        },
        data() {
            return {
                fileInput: {},
                terms: [],
                sourceList: [],
                interlexTerms: [],
                selectedTerm: null ,
                selectedFile: null,
                image: { backgroundImage: "static/images/nidm-terms-background.png" },
                showForm: false,
                responses: {},
                concepts: {},
                completed: {},
                isAnnotated: false,
                showBroadenButton: false,
                broadenConceptSearch: false,
                selectedConcepts: [],
                initSearch: ''
            }
        },
        watch: {
            responses: {
                deep: true,
                handler(newVal) {
                    this.isAnnotated = !_.isEmpty(newVal);
                    // eslint-disable-next-line
                    // console.log(82, 'in resp watcher ', newVal, _.isEmpty(newVal), this.isAnnotated);

                }
            },
            concepts: {
                deep: true,
                // eslint-disable-next-line no-unused-vars
                handler(newVal) {
                    // eslint-disable-next-line
                    // console.log(103, 'in concepts watcher ', newVal);
                    this.$forceUpdate();
                }
            },
            interlexTerms() {
                this.$forceUpdate();
            },
            completed: {
                deep: true,
                // eslint-disable-next-line no-unused-vars
                handler(newVal) {
                    // eslint-disable-next-line
                    // console.log(92, 'in completed watcher ', this.selectedTerm, newVal);
                }
            },
        },
        methods: {
            downloadJSONLD() {
                const annotatedFile = this.createAnnotatedFile(this.responses);
                const fileName = `${this.fileInput.name.split('.')[0]}.json`;
                // Create a blob of the data
                const fileToSave = new Blob([JSON.stringify(annotatedFile, null, 4)], {
                    type: 'application/json',
                    name: fileName
                });
                // Save the file
                saveAs(fileToSave, fileName);
            },
            createAnnotatedFile(responses) {
                // eslint-disable-next-line
                // console.log(117, responses);
                // const r = {
                //     "race": {
                //         "label": "race1",
                //         "sourceVariable": "race",
                //         "description": "something",
                //         "url": "",
                //         "valueType": "",
                //         "isAbout": [],
                //         "isPartOf": [],
                //         "datumType": "",
                //         "unitCode": "",
                //         "maxValue": "",
                //         "minValue": "",
                //         "allowableValues": [],
                //         "choices": [],
                //         "derivative": "",
                //         "citation": "",
                //         "measureOf": "",
                //         "provenance": ""
                //     }
                // };
                let annotatedObject = {};

                for (const [key, value] of Object.entries(responses)) {
                    let responseOptions = {};

                    const annotateKey = `DD(source=${this.fileInput.name}, variable=${key})`;
                    const annotatedValue = {
                        "label": key,
                        "description": value.description,
                        "sourceVariable": value.sourceVariable,
                        "isAbout": value.isAbout
                    };
                    responseOptions.valueType = `xsd:${value.valueType}`;
                    if (value.hasUnit) {
                        responseOptions.hasUnit = value.hasUnit;
                    }
                    if (value.minValue !== '') {
                        responseOptions.minValue = value.minValue;
                    }
                    if (value.maxValue !== '') {
                        responseOptions.maxValue = value.maxValue;
                    }
                    if (value.choicesName.length && value.choicesValue.length) {
                        let choices_list = [];
                        value.choicesName.forEach((num1, index) => {
                            const num2 = value.choicesValue[index];
                            choices_list.push({'name': num1, 'value': num2});
                        });
                        responseOptions.choices = choices_list;
                    }
                    if (!_.isEmpty(responseOptions)) {
                        annotatedValue.responseOptions = responseOptions;
                    }
                    annotatedObject[annotateKey] = annotatedValue;
                }
                return annotatedObject;
            },
            async test_enter_handler(event) {
                // eslint-disable-next-line no-console
                // console.log(199, event.target.value);
                this.initSearch = event.target.value;
                // this.activeIndex = index;
                const term = event.target.value;
                this.$emit('termSelect', term);
                // search for concepts in NIDM_Concepts first
                const resp = await axios.get('https://api.github.com/repos/NIDM-Terms/terms/contents/terms/NIDM_Concepts');
                const matching_names = _.filter(resp.data, concept => concept.name.search(term) !== -1);

                if (matching_names.length) {
                    const promises = matching_names.map(async (match_term) => {
                        const res = await axios.get(match_term.download_url);
                        return res.data;
                    });

                    const resolved = await Promise.all(promises);
                    // this.$emit('termSearchResult', resolved);
                    this.searchList(resolved);
                    this.$emit('nidmConceptFound', true);
                    // eslint-disable-next-line no-console
                    console.log('NIDM concepts found!!!! ', resolved);
                }
                else {
                    this.$emit('nidmConceptFound', false);
                    // eslint-disable-next-line no-console
                    // console.log('NIDM concepts NOT found!!!! ');
                    // since no matching nidm concepts, now search interlex
                    axios.get(`https://scicrunch.org/api/1/elastic/interlex/term/_search?q=${term}`, {
                        params: {
                            key: API_KEY
                            // source: JSON.stringify(query),
                            // source_content_type: 'application/json'
                        }
                    }).then(response => {
                        // eslint-disable-next-line
                        // console.log(56, term, response.data.hits.hits);
                        const m_list = _.map(response.data.hits.hits, match1  => {
                            const ilx_id = _.find(match1['_source']['existing_ids'], (id) => {
                                return id.curie.startsWith('ILX:');
                            });
                            return { 'label': match1['_source']['label'],
                                'description': match1['_source']['definition'],
                                'url': ilx_id.iri}
                        });
                        // this.$emit('termSearchResult', m_list);
                        this.searchList(m_list);
                        // eslint-disable-next-line no-console
                        // console.log('interlex concepts found!!! ', m_list);
                    });
                }
            },
            onFileSelected(event) {
                this.fileInput = event.target.files[0];
                // eslint-disable-next-line
                // console.log(116, this.fileInput.name.split('.')[0]);
                Papa.parse(this.fileInput, {
                    complete: results => {
                        this.sourceList = results.data[0];
                    },

                });
            },
            searchList(termList) {
                // eslint-disable-next-line no-console
                // console.log(164, termList);
                this.interlexTerms = termList;
            },
            showTermProperties(termSelected) {
                this.selectedTerm = termSelected;
                this.showForm = true;
            },
            showBroaden(showB) {
                this.showBroadenButton = showB;
            },
            broadenSearchList() {
                this.broadenConceptSearch = true;
            },
            updateConceptList(val) {
                this.selectedConcepts = val;
            },
            setResponse(selectedTerm, annotations) {
                this.$set(this.responses, selectedTerm, annotations);
                // eslint-disable-next-line
                // console.log(104, selectedTerm, annotations, isAboutList);
                // this.responses[selectedTerm] = annotations;
                // todo: check if all properties are set and then set completed to true or false
                if (annotations) { //change the condition
                    this.$set(this.completed, selectedTerm, true);
                    // eslint-disable-next-line
                    // console.log(128, 'in setResponse set cond', this.completed, Object.values(annotations));
                } else this.$set(this.completed, selectedTerm, false);
            },
            setConcepts(selectedTerm, selectedConcepts) {
                this.$set(this.concepts, selectedTerm, selectedConcepts);
            }
        },

    }
</script>

<style scoped>
    #annotate {
        font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
        /*background-color: rgba(199, 176, 211, 0.71);*/
    }
    .col-container {
        display: flex;
        width: 100%;
        padding-bottom: 20px;
    }

    .col-header {
        /*background-color: #009DAA;*/
        /*height: 60px;*/
        background-color: lightsteelblue;
        /*border-style: solid;*/
        /*border-color: red;*/
        border-bottom: 1px solid grey;
        text-align: center;
    }
    .col-header > p {
        color: #444;
        padding-top: 15px;
    }

    .header-text {
        color: #444;
        padding-top: 15px;
    }

    .concept-search {
        color: #444;
        padding-top: 11px;
        align-self: ;
    }

    .bv-example-row {
        border: black;
    }
    /*Demo Styles*/

    .upload {
        font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
        text-align: left;
        color: #444;
        padding-top: 20px;
        padding-left: 20px;
        /*padding: 150px 0;!*Add some height to the columns*!*/
    }

    input {
        padding-left: 15px;
        padding-bottom: 15px;
        border-radius: 8px;
        width: calc(33.333333% - 20px);
        box-sizing: border-box;
    }

    input:hover {
        cursor: pointer;
    }

    #main-content {
        /*background-color: #eef3f7;*/
        width: 50%;
        float: left;
        border-top: 1px solid grey;
        border-bottom: 1px solid grey;
    }

    #sidebar-left {
        /*background-color: #e3e9ee;*/
        width: 25%;
        float: left;
        border: 1px solid grey;
    }

    #sidebar-right {
        /*background-color: #dbe1e6;*/
        width: 25%;
        float: left;
        border: 1px solid grey;
    }

    /*Base Mobile Layout*/
    .wrap {
        width: 90%;
        margin: 0 auto;
    }

    /*3 Column Layout*/
    /*@media only screen and (min-width: 1024px) {*/
    /*    .wrap {*/
    /*        width: 1024px;*/
    /*        margin: 0 auto;*/
    /*    }*/

    /*    #main-content {*/
    /*        width: 50%;*/
    /*        float: left;*/
    /*    }*/

    /*    #sidebar-left {*/
    /*        width: 25%;*/
    /*        float: left;*/
    /*    }*/

    /*    #sidebar-right {*/
    /*        width: 25%;*/
    /*        float: left;*/
    /*    }*/
    /*}*/

    /*Wide Layout*/
    /*@media only screen and (min-width: 1200px) {*/
    /*    .wrap {*/
    /*        width: 1140px;*/
    /*        margin: 0 auto;*/
    /*    }*/
    /*}*/

    /*Move Sidebar Position*/
    /*@media only screen and (min-width: 1024px) {*/
    /*    .wrap, #main-content, #sidebar-left, #sidebar-right {*/
    /*        position: relative;*/
    /*    }*/

    /*    #main-content, #sidebar-left, #sidebar-right {*/
    /*        top: 0;*/
    /*    }*/

    /*    #sidebar-right {*/
    /*        right: 0;*/
    /*    }*/

    /*    #sidebar-left {*/
    /*        left: -50%; !*Width of #main-content*!*/
    /*    }*/

    /*    #main-content {*/
    /*        left: 25%; !*Width of #left-sidebar*!*/
    /*    }*/
    /*}*/

    /*Optional 2 Column Layout*/
    /*@media only screen and (min-width: 768px) and (max-width: 1024px) {*/
    /*    #sidebar-right {*/
    /*        width: 50%;*/
    /*    }*/
    /*    #sidebar-left {*/
    /*        width: 50%;*/
    /*    }*/
    /*    #main-content {*/
    /*        position: relative;*/
    /*        top: 0;*/
    /*        left: 0;*/
    /*    }*/
    /*}*/

    .community-logo {
        height: 60px;
        margin-right: 10px;
        vertical-align: middle;
    }

    .community-name {
        font-size: 28px;
    }

    .breadcrumbs-v3 {
        padding: 10px 0;
        background: #585f69;
        border-bottom: 1px solid #eee;
    }

    .breadcrumb {
        top: 10px;
        padding-right: 0;
        background: none;
        position: relative;
    }

    .pull-right {
        float: right!important;
    }

    .download-btn {
        padding-top: 10px;
        padding-bottom: 20px;
        padding-left: 15px;
    }

</style>
