<template>
   <div>
       <v-card>
        <v-card-title class="headline">
            Search By Name
        </v-card-title>
        <v-card-text>
             <!-- QUERY TO LOAD ROUTE NAMES FOR AUTOSUGGESTION -->
            <!-- NAME IS THEN USED BY ANOTHER QUERY GET SPECIFIC ROUTE ID ETC -->
            <ApolloQuery
            :query="require('../graphql/AllRoutes.gql')"
            >
            <template v-slot="{ result: { loading, error, data } }">
                <!-- Loading -->
                <div v-if="loading" class="loading apollo">Loading...</div>

                <!-- Error -->
                <div v-else-if="error" class="error apollo">
                        <v-list-item>
                        <v-icon color="white" class="pr-2">mdi-alert-circle</v-icon>
                        <v-list-item-content>An Error Occured...</v-list-item-content>
                    </v-list-item>
                </div>

                <!-- Result -->
                <div v-else-if="data" class="result apollo">  
                        <v-autocomplete
                        v-model="searchString"
                        :items= "data.Routes.map(route => route.name)"
                        color="blue"
                        label="Routes"
                        placeholder="Start typing to Search"
                        prepend-icon="mdi-database-search"
                        class="mt-2 mb-0"
                    ></v-autocomplete>
                </div>
                <!-- No result -->
                <div v-else class="no-result apollo">No result :(</div>
            </template>
            </ApolloQuery>
        </v-card-text>
        <v-divider></v-divider>
             <!-- SECTION EXPANDS BASED ON "SEARCHSTRING" BEING ENTERED -->    
            <!-- EXPANDED SECTION CONTAINS REMAINING FORM INPUTS FOR ROUTE UPDATE -->
            <v-expand-transition>
            <v-list v-if="searchString">

             <!-- USES "SEARCHSTRING" TO SEARCH FOR SPECIFIC ROUTE -->
            <!-- ONLY RETURNS NEEDED ROUTE FOR POPULATING ID INTO THE FORM FOR USER -->
                <ApolloQuery
                :query="require('../graphql/SearchRoutes.gql')"
                :variables="{ searchString }"
                >
                    <template v-slot="{ result: { loading, error, data } }">

                        <div v-if="loading" class="loading apollo">
                            <v-list-item>
                                <v-icon color="white" class="pr-2">mdi-arrow-down-bold-circle-outline</v-icon>
                                <v-list-item-content>Loading...</v-list-item-content>
                            </v-list-item>
                        </div>

                        <div v-else-if="error" class="error apollo">
                            <v-list-item>
                                <v-icon color="white" class="pr-2">mdi-alert-circle</v-icon>
                                <v-list-item-content>An Error Occured...</v-list-item-content>
                            </v-list-item>
                        </div>

                        <div v-else-if="data" class="result apollo">
                            <v-list-item
                            v-for="(route, i) in data.Routes" 
                            :key="i"
                            >
                            <v-list-item-content>
                                <v-list-item-title> {{  route.name }}</v-list-item-title>
                                <v-list-item-subtitle>Distance: {{  route.miles }} miles</v-list-item-subtitle>
                                <v-list-item-subtitle>Starting Elevation {{  route.startingElevation }} ft.</v-list-item-subtitle>
                                <v-list-item-subtitle>Final Elevation {{  route.finalElevation }} ft.</v-list-item-subtitle>
                                <v-list-item-subtitle>Elevation Gain {{  route.finalElevation - route.startingElevation }} ft.</v-list-item-subtitle>
                                <v-list-item-subtitle>Average Grade: {{ (((route.finalElevation - route.startingElevation) / (route.miles*5280)) * 100).toFixed(1)  }}%</v-list-item-subtitle>
                                <v-list-item-subtitle>ID: {{route.id}}</v-list-item-subtitle>
                                <iframe
                                width="100%"
                                height="350"
                                :src= "route.iframeData"
                                class="mt-2"
                                >
                                </iframe>
                            </v-list-item-content>
                            </v-list-item>
                        </div>

                        <div v-else class="no-result apollo">
                            <v-list-item>
                                <v-icon color="white" class="pr-2">mdi-clipboard-alert</v-icon>
                                <v-list-item-content>No Results...</v-list-item-content>
                            </v-list-item>
                        </div>
                    </template>
                </ApolloQuery>
            </v-list>
            <v-expand-transition></v-expand-transition>
            </v-expand-transition>
            <v-card-actions>
                <v-spacer></v-spacer>
                    <v-btn
                    :disabled="!searchString"
                    color="#EA4235"
                    outlined
                    @click="searchString = null"
                    >
                    Clear
                    <v-icon right>mdi-close-circle</v-icon>
                </v-btn>
            </v-card-actions>
       </v-card>
    </div>
</template>

<script>

  export default {
    data: () => ({
    searchString: '',
    }),
  }
</script>
