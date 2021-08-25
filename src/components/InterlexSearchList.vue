<template>
    <div>
        <div v-if="terms.length">
            <ul class="list-group scroll" v-for="term in terms"
                :key="term['_id']">
                <li class="list-group-item">
                    <input type="checkbox" class="checkbox" :value="term.url"
                           v-model="checkedCategories"
                           @change="onTermSelect">
                    <span class="align-middle">
                        <strong>{{ term['label'] }} </strong>
                        <br>
                        Description: {{ term['description'] }}
                    </span>
                </li>
            </ul>
            <b-button v-if="broaden" variant="outline-primary" class="broaden" @click="broadenSearch">Broaden Search</b-button>
        </div>
        <div v-else>
            <p class="no-match">No matching terms found!</p>
        </div>
    </div>
</template>

<script>
    import axios from "axios";
    import _ from 'lodash';
    const API_KEY = 'CfufpoSNVXujv7h14HFHI4dL9p36mxCJ';
    export default {
        name: "InterlexSearchList",
        props: {
            terms: Array,
            broaden: Boolean,
            selectedTerm: String,
            init: Array,
        },
        data() {
            return {
                concepts: [],
                checkedCategories: [],
                mainCategories: [{
                    merchantId: '1'
                }, {
                    merchantId: '2'
                }]
            }
        },
        beforeMount() {
            this.concepts = this.terms;
        },
        methods: {
            onTermSelect() {
                // this.checkedCategories.push(val);
                // eslint-disable-next-line no-console
                this.$emit('termSelect', this.checkedCategories);
                this.$emit('saveCheckedConcepts', this.selectedTerm, this.checkedCategories);
            },
            broadenSearch() {
                // eslint-disable-next-line no-console
                console.log(39, this.selectedTerm);
                // this.$emit('broadenSearch');
                axios.get(`https://scicrunch.org/api/1/elastic/interlex/term/_search?q=${this.selectedTerm}`, {
                    params: {
                        key: API_KEY
                        // source: JSON.stringify(query),
                        // source_content_type: 'application/json'
                    }
                }).then(response => {
                    // eslint-disable-next-line
                    console.log(54, this.selectedTerm, response.data.hits.hits);
                    const m_list = _.map(response.data.hits.hits, match1  => {
                        // eslint-disable-next-line no-console
                        console.log(113, match1);
                        return { 'label': match1['_source']['label'], 'description': match1['_source']['definition']}
                    });
                    this.terms = m_list;
                    // this.$emit('termSearchResult', m_list);
                    // eslint-disable-next-line no-console
                    console.log('interlex concepts found!!! after broaden');
                });
            }
        },
        mounted() {
            if (this.init) {
                this.checkedCategories = this.init;
            }
        }
    };
</script>

<style scoped>
    ul {
        /*top: 20px;*/
        /*bottom: 20px;*/
        /*left: 20px;*/
        /*right: 20px;*/
        /*overflow: scroll;*/
        /*margin: 0;*/
        /*padding: 0;*/

        font-size: 16px;
        height: auto;
        /*-webkit-overflow-scrolling: touch;*/
    }

    .scroll {
        /*margin:4px;*/
        /*padding:4px;*/

        overflow-y: scroll;
    }

    .no-match {
        margin: 20px;
        text-align: center;
    }

    .broaden {
        float: right;
    }
    .broaden.btn-outline-primary {

        border-color: transparent;
    }
    .broaden.btn-outline-primary:hover {
        background-color: transparent;
        color: blue;
    }

    li {
        display: flex;
        /*cursor: pointer;*/
    }

    li:hover {
        background-color: #eee;
    }
    .checkbox{
        margin: 10px!important;
        width: auto!important;
    }
</style>
