<template>
    <div>
        <ul class="list-group">
            <!-- key property is used whenever list is updated -->
<!--            <TermListItem-->
<!--                    v-for="term in terms1"-->
<!--                    :term="term"-->
<!--                    :key="term"-->
<!--                    :completed="completed"-->
<!--                    @termSelect="onTermSelect"-->
<!--            >-->
<!--            </TermListItem>-->

            <li class="list-group-item" v-for="(term , idx) in terms1" :key="idx+term"
                :class="{'current-item': idx === activeIndex}" @click="onTermSelect(term, idx)">

                    <span>
                     {{term}}
                   </span>

                <div v-if="completed[term]" class="checkmark"></div>
            </li>
        </ul>
    </div>
</template>

<script>
    import axios from "axios";
    import _ from 'lodash';
    const API_KEY = 'CfufpoSNVXujv7h14HFHI4dL9p36mxCJ';

    export default {
        name: "SourceVariableList",
        props: {
            terms1: Array,
            completed: Object,
            broadenSearch: Boolean,
        },
        data() {
          return {
              results: [],
              itemCompleted: false,
              activeIndex: null,
              nidmConceptFound: false,
          }
        },
        methods: {
            async onTermSelect(term, index) {
                // eslint-disable-next-line
                // console.log(47, term, index);
                this.activeIndex = index;
                this.$emit('termSelect', term);
                // const query = {
                //     query: {
                //         bool: {
                //             must : [
                //                 {  term : { type : "cde" } },
                //                 { term : { "ancestors.ilx" : "ilx_0115066" } },
                //                 { multi_match : {
                //                         query: term,
                //                         fields: [ "label" , "definition" ]
                //                     }
                //                 }
                //             ]
                //         }
                //     }
                // };
                // search for concepts in NIDM_Concepts first
                const resp = await axios.get('https://api.github.com/repos/NIDM-Terms/terms/contents/terms/NIDM_Concepts');
                // eslint-disable-next-line no-console
               // console.log(78, 'response is: ', resp.data);
               const matching_names = _.filter(resp.data, concept => concept.name.search(term) !== -1);
                // eslint-disable-next-line no-console
               // console.log(71, matching_names);

                if (matching_names.length) {
                    const promises = matching_names.map(async (match_term) => {
                        const res = await axios.get(match_term.download_url);
                        return res.data;
                    });

                    const resolved = await Promise.all(promises);
                    this.$emit('termSearchResult', resolved);
                    this.$emit('nidmConceptFound', true);
                    // eslint-disable-next-line no-console
                    console.log('NIDM concepts found!!!! ', resolved);
                }

               else {
                    this.$emit('nidmConceptFound', false);
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
                       this.$emit('termSearchResult', m_list);
                       // eslint-disable-next-line no-console
                       console.log('interlex concepts found!!! ', m_list);
                   });
               }



            },
        },
    };
</script>

<style scoped>
    .checkmark {
        display: inline-block;
        transform: rotate(45deg);
        height: 25px;
        width: 12px;
        margin: auto;
        margin-right: inherit;
        border-bottom: 4px solid #78b13f;
        border-right: 4px solid #78b13f;
    }

    li {
        display: flex;
        cursor: pointer;
    }

    .current-item {
        background-color: lightgrey;
    }
</style>
