<template>
    <div id="app">
        <form @submit="e => e.preventDefault()">
            <input type="text" v-model="query" placeholder="Input anything in any language..."/>
            <button type="button" @click="guessLingo">Try me!</button>
        </form>

        <h3 v-if="this.loading">Loading...</h3>

        <div class="result">
            <h2 v-if="result.language">
                {{ result.language }} -
                {{ result.probability }}%
                ({{ result.reliability ? '&check;' : '&cross;' }})
            </h2>
            <ul v-if="result.list.length > 1">
                <li v-for="row in result.list" :key="row.language_code">
                    {{ row.language_name }} -
                    {{ row.percentage.toFixed(2) }}%
                    ({{ row.reliable_result ? '&check;' : '&cross;' }})
                </li>
            </ul>
        </div>
    </div>
</template>

<script>
    import axios from 'axios'

    export default {
        name: 'app',
        data() {
            return {
                loading: false,
                query: '',
                api_key: '',
                api_url: 'http://apilayer.net/api/detect',
                result: {
                    language: '',
                    probability: 0.0,
                    reliability: false,
                    list: [],
                }
            }
        },
        computed: {
            url() {
                return `${this.api_url}?access_key=${this.api_key}&query=${this.query}`;
            }
        },
        methods: {
            getMostProbable(results, porperty) {
                let allProps = results.map(o => o[porperty]),   // Get array of `property`
                    greatest = Math.max(...allProps),           // Get element of which `property` is greatest
                    indexGreatest = allProps.indexOf(greatest); // Get index from element

                // Return element with greatest `property` value
                return results[indexGreatest]
            },
            guessLingo() {
                this.loading = true;
                axios.get(this.url)
                    .then(response => {
                        // Hide load indicator
                        this.loading = false;

                        // Destructure result with highest probability value
                        let {language_name, percentage, reliable_result} = this.getMostProbable(response.data.results, 'probability');

                        // Shift most probable off result
                        response.data.results.shift();

                        // Set result
                        this.result = {
                            list: response.data.results,
                            language: language_name,
                            probability: percentage.toFixed(2),
                            reliability: reliable_result
                        }
                    }, error => console.error(error));
            }
        }
    }
</script>

<style>
    body, html, form, input {
        margin: 0;
        padding: 0;
    }

    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #000;
        display: flex;
        flex-flow: column wrap;
    }

    h2 {
        background-color: lightgreen;
        margin: 0;
        padding: 1rem;
    }

    ul {
        margin: 0;
        padding: 1rem;
        list-style-type: none;
        background: lightblue;
    }

    form {
        display: flex;
        flex-flow: row nowrap;
    }

    form > input {
        flex: 6;
        padding: 1rem;
        font-size: 1.33rem;
        letter-spacing: 0.05rem;
    }

    form > button {
        flex: 1;
    }
</style>
