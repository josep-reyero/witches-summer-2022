<template>
  <div id="outer">
    <div id="inner">
      <div id="page-intro" class="pl-5 pr-5 pt-3 pb-3">
        <div class="flex content-start items-center">
          <h1 class="text-sm md:text-xl lg:text-2xl">Places of Residence for Accused Witches With Timeline <template v-if="noItems > 0">(total named accused witches: 3141)</template>
          </h1>
          <span class="rounded-full border-r border-l border-gray-400 w-6 h-6 flex items-center justify-center ml-2">
            <!-- icon by feathericons.com -->
            <svg aria-hidden="true" class="" data-reactid="266" fill="none" height="24" stroke="#606F7B" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg" @click="showPageInfo()">
              <line x1="12" y1="16" x2="12" y2="12"></line><line x1="12" y1="8" x2="12" y2="8"></line>
            </svg>
          </span>
        </div>
        <div id="map-filters" :class="filters ? 'block': 'hidden'" class="pt-2">
          <div>
            <span v-for="(tile, index) in tiles">
              <input type="radio" name="tile" :checked="tile.name === currentTileName" @change="filterTiles(tile)"/>&nbsp;{{tile.name}}&nbsp;
            </span>
          </div>
          <br>
          <div>
            <span v-for="(layer, index) in layers">
              <input type="radio" name="layer" :checked="layer.id === currentLayer.id" @change="filterLayers(layer)"/>&nbsp;{{layer.label}}&nbsp;
            </span>
          </div>
          <br>
          <div v-if="currentLayer.id === 'sexes'" >
            <span v-for="(sex, index) in sexes" class="flex items-center float-left">
              <input type="checkbox" v-model="sex.active" @change="filterMarkers()"/>&nbsp;<img :src="sex.iconUrl" width="12" height="20"/>&nbsp;{{sex.type}}&nbsp;
            </span>
          </div>
          <div v-if="currentLayer.id === 'occupations'" >
            <span v-for="(occupation, index) in occupations" class="flex items-center float-left">
              <input type="checkbox" v-model="occupation.active" @change="filterMarkers()"/>&nbsp;<img :src="occupation.iconUrl" width="12" height="20"/>&nbsp;{{occupation.type}}&nbsp;
            </span>
          </div>
          <div v-if="currentLayer.id === 'socials'" >
            <span v-for="(social, index) in socials" class="flex items-center float-left">
              <input type="checkbox" v-model="social.active" @change="filterMarkers()"/>&nbsp;<img :src="social.iconUrl" width="12" height="20"/>&nbsp;{{social.type}}&nbsp;
            </span>
          </div>
          <div v-if="currentLayer.id === 'wikis'" >
            <span v-for="(wiki, index) in wikis" class="flex items-center float-left">
              <input type="checkbox" v-model="wiki.active" @change="filterMarkers()"/>&nbsp;<img :src="wiki.iconUrl" width="12" height="20"/>&nbsp;{{wiki.type}}&nbsp;
            </span>
          </div>
          <br><br>
          <h2>Year: {{ sliderYear[0] }} - {{ sliderYear[1] }}</h2>
          <div class="p-2">
            <vue-slider v-model="sliderYear" :adsorb="false" :data="sliderYears" :marks="true" @change="filterMarkers()"></vue-slider>
          </div>
        </div>
      </div>
      <div class="border border-gray p-1 bg-gray-200" v-if="!loading">
        <span class="flex items-center float-left">
          &nbsp;Filters
        </span>
        <span class="rounded-full border-r border-l border-gray-400 w-6 h-6 flex items-center justify-center ml-2 float-left">
          <!-- icon by feathericons.com -->
          <svg v-if="!filters" aria-hidden="true" class="" data-reactid="266" fill="none" height="24" stroke="#606F7B" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg" @click="toggleFilters()">
            <polyline points="6 9 12 15 18 9">
            </polyline>
          </svg>
          <svg v-if="filters" aria-hidden="true" class="" data-reactid="266" fill="none" height="24" stroke="#606F7B" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg" @click="toggleFilters()">
            <polyline points="18 15 12 9 6 15">
            </polyline>
          </svg>
        </span>
      </div>
      <div id="map-wrapper">
        <div class="mx-auto max-w-md bg-white rounded shadow-md mt-10" v-if="loading">
          <div class="pt-8 pb-12 pl-8 pr-8">
            <div class="float-left align-text-bottom">Loading map&nbsp</div>
            <div class="lds-facebook float-left"><div></div><div></div><div></div></div>
          </div>
        </div>
        <no-ssr v-else>
          <l-map style="height: 100%; width: 100%" :zoom="zoom" :center="center" ref="myMap">
            <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
            <!-- <v-marker-cluster ref="clusterRef" :options="clusterOptions"> -->
            <l-marker v-for="(marker, index) in activeMarkers"
                      :lat-lng="marker.longLat">
              <l-popup class="adapted-popup">
                <h2>{{marker.location}}</h2><br>
                <div :class="marker.witches.length > 1 ? 'witch-scroller' : 'no-witch-scroller'">
                  <div v-for="(witch, index) in marker.witches">
                    <strong>{{ witch.name }}</strong><br>
                    Investigation Date: {{ witch.investigationDate }}<br>
                    Gender: {{ witch.sex }}<br>
                    Occupation: {{ witch.occupation }}<br>
                    Social Class: {{ witch.socialClassification }}<br>
                    <div v-if="witch.residences.length > 0">
                      Residences:
                      <template v-for="(residence, index) in witch.residences">
                        <a @click="flyTo(residence.coords)" :style="{ cursor: 'pointer'}">{{ residence.location }}</a><template v-if="index < witch.residences.length - 1">, </template>
                      </template>
                      <br>
                    </div>
                    <div v-if="witch.detentions.length > 0">
                      Places of Detention:
                      <template v-for="(detention, index) in witch.detentions">
                        <a @click="flyTo(detention.coords)" :style="{ cursor: 'pointer'}">{{ detention.location }}</a><template v-if="index < witch.detentions.length - 1">, </template>
                      </template>
                      <br>
                    </div>
                    <div v-if="witch.placeOfDeath !== ''">
                      Place of Death: <a @click="flyTo(witch.placeOfDeathCoords)" :style="{ cursor: 'pointer'}">{{ witch.placeOfDeath }}</a><br>
                    </div>
                    <div v-if="witch.mannerOfDeath !== ''">
                      Manner of Death: {{ witch.mannerOfDeath }}<br>
                    </div>
                    <div v-if="witch.wikiPage !== ''">
                      <a :href="witch.wikiPage" target="_blank">View Wiki Page</a><br>
                    </div>
                    <a :href="witch.link" target="_blank">More Info</a><br><br>
                  </div>
                </div>
              </l-popup>
              <l-icon :icon-anchor="iconAnchor" :key="marker">
                <div class="icon-wrapper">
                  <div v-if="hasWikiEntry(marker)" class="icon-wiki">W</div>
                  <div v-if="marker.witches.length > 1" class="icon-text">{{marker.witches.length}}</div>
                  <img :src="getIcon(marker)" class="zoomed-in-img"/>
                  <img class="icon-shadow" :src="shadowUrl"/>
                </div>
              </l-icon>
            </l-marker>
            <!-- </v-marker-cluster> -->
          </l-map>
        </no-ssr>
      </div>
    </div>
  </div>
</template>

<script>

 import {SPARQLQueryDispatcher} from '~/assets/js/SPARQLQueryDispatcher';

 export default {
   data: () => ({
     loading: true,
     filters: false,
     noItems: 0,
     sparqlUrl: 'https://query.wikidata.org/sparql',
     url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
     attribution: 'Map data © <a href="https://www.openstreetmap.org/copyright">OpenStreetMap contributors</a>. Historical Maps Layer, 1919-1947 from the <a href="http://maps.nls.uk/projects/api/">NLS Maps API</a>',
     zoom: 7,
     center: [55.95, -3.198888888],
     clusterOptions: {
       iconCreateFunction: function (cluster) {
         var html = '<div><img src="/images/witches-cluster-composite-yellow.png" width="72" height="54"></div>';
         return L.divIcon({ html: html, className: 'mycluster', iconSize: L.point(72, 54) });
       },
       disableClusteringAtZoom : 12,
       spiderfyOnMaxZoom: false
     },
     sliderYear: [1550, 1750],
     sliderYears: [1550, 1575, 1600, 1625, 1650, 1675, 1700, 1725, 1750],
     wikiPages: [],
     markers: [],
     originalMarkers: [],
     currentTileName : 'Modern Map',
     tiles: [{name: 'Modern Map', url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', active: true},{name: 'Historic Map', url: 'https://nls.tileserver.com/nls/{z}/{x}/{y}.jpg', active : false}],
     layers: [{id: 'sexes', label: 'Gender', property : 'sex'}, {id: 'socials', label: 'Social Classification', property : 'socialClassification'}, {id: 'occupations', label: 'Occupations', property : 'occupation'},  {id: 'wikis', label: 'Wikipedia Page', property : 'hasWikiPage'}],
     currentLayer : {id: 'sexes', label: 'Gender', property : 'sex'},
     sexes: [{type: 'male', active: true, iconUrl: '/images/witch-single-blue.png'},{type: 'female', active: true, iconUrl: '/images/witch-single-orange.png'}, {type: 'unknown', active: true, iconUrl: '/images/witch-single-BW.png'}],
     wikis: [{type: 'has wiki', active: true, iconUrl: '/images/witch-single-blue.png'},{type: 'no wiki', active: true, iconUrl: '/images/witch-single-orange.png'}],
     socials: [],
     occupations: [],
     icons: ['/images/witch-single-blue.png',
             '/images/witch-single-orange.png',
             '/images/witch-single-pink.png',
             '/images/witch-single-red.png',
             '/images/witch-single-brown.png',
             '/images/witch-single-green.png',
             '/images/witch-single-pale-blue.png',
             '/images/witch-single-yellow.png',
             '/images/witch-single-blue.png',
             '/images/witch-single-orange.png',
             '/images/witch-single-pink.png',
             '/images/witch-single-red.png',
             '/images/witch-single-brown.png',
             '/images/witch-single-green.png',
             '/images/witch-single-pale-blue.png',
             '/images/witch-single-yellow.png',
             '/images/witch-single-blue.png',
             '/images/witch-single-orange.png',
             '/images/witch-single-pink.png',
             '/images/witch-single-red.png',
             '/images/witch-single-brown.png',
             '/images/witch-single-green.png',
             '/images/witch-single-pale-blue.png',
             '/images/witch-single-yellow.png',
             '/images/witch-single-blue.png',
             '/images/witch-single-orange.png',
             '/images/witch-single-pink.png',
             '/images/witch-single-red.png',
             '/images/witch-single-brown.png',
             '/images/witch-single-green.png',
             '/images/witch-single-pale-blue.png',
             '/images/witch-single-yellow.png',
             '/images/witch-single-blue.png',
             '/images/witch-single-orange.png',
             '/images/witch-single-pink.png',
             '/images/witch-single-red.png',
             '/images/witch-single-brown.png',
             '/images/witch-single-green.png',
             '/images/witch-single-pale-blue.png',
             '/images/witch-single-yellow.png',
             '/images/witch-single-blue.png',
             '/images/witch-single-orange.png',
             '/images/witch-single-pink.png',
             '/images/witch-single-red.png',
             '/images/witch-single-brown.png',
             '/images/witch-single-green.png',
             '/images/witch-single-pale-blue.png',
             '/images/witch-single-yellow.png',
     ]
   }),
   methods: {
     convertPointToLongLatArray: function(pointString) {
       pointString = pointString.substr(6);
       pointString = pointString.slice(0,-1);
       let pointArray = pointString.split(' ');
       let longLatArray = [pointArray[1], pointArray[0]];
       return longLatArray;

     },
     loadWikiEntries : function() {
       const sparqlQuery = `SELECT DISTINCT ?item ?LabelEN ?page_title
            WHERE {
              ?item wdt:P4478 ?witch .
              ?article schema:about ?item ; schema:isPartOf <https://en.wikipedia.org/> ;  schema:name ?page_title .
              ?item rdfs:label ?LabelEN filter (lang(?LabelEN) = "en") .
            }`;

       const queryDispatcher = new SPARQLQueryDispatcher( this.sparqlUrl );
       queryDispatcher.query( sparqlQuery ).then( result => {

         for (let i = 0; i < result.results.bindings.length; i++) {
           let item = result.results.bindings[i];

           let wikiPage = {
             id: item.item.value,
             pageTitle: item.page_title.value,
           }

           this.wikiPages.push(wikiPage);
         }

         this.loadAccussed();
       });
     },
     loadAccussed : function() {

       const sparqlQuery = `SELECT distinct ?item ?itemLabel ?investigationDate
            ?residenceLabel ?residenceCoords ?sexLabel ?link ?occupationLabel ?socialClassificationLabel
            ?placeOfDeathLabel ?placeOfDeathCoords ?mannerOfDeathLabel ?detentionLocationLabel ?detentionLocationCoords
            WHERE
            {
              ?item wdt:P4478 ?witch .
              ?item wdt:P551 ?residence .
              ?residence wdt:P625 ?residenceCoords .
              optional { ?item wdt:P21 ?sex } .
              ?item wdt:P4478 ?link .
              optional { ?item wdt:P106 ?occupation .}
              optional { ?item wdt:P3716 ?socialClassification .}
              optional {
                ?item wdt:P20 ?placeOfDeath .
                ?placeOfDeath wdt:P625 ?placeOfDeathCoords .}
              optional { ?item wdt:P1196 ?mannerOfDeath .}
              optional { ?item p:P793 ?significantEventStatement .
              ?significantEventStatement ps:P793 wd:Q66458810 .
              OPTIONAL {?significantEventStatement pq:P585 ?investigationPoint }.
              OPTIONAL {?significantEventStatement pq:P580 ?investigationStart }
              }
              BIND(IF(BOUND(?investigationPoint), ?investigationPoint, ?investigationStart) as ?investigationDate)
              optional {  ?item wdt:P2632 ?detentionLocation .
                ?detentionLocation wdt:P625 ?detentionLocationCoords .}

              SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
            }`;

       const queryDispatcher = new SPARQLQueryDispatcher( this.sparqlUrl );
       queryDispatcher.query( sparqlQuery ).then( result => {

         let witches = [];

         for (let i = 0; i < result.results.bindings.length; i++) {

           let item = result.results.bindings[i];
           let id = item.item.value;
           let sex = item.hasOwnProperty('sexLabel') ? item.sexLabel.value : 'unknown';
           let occupation = item.hasOwnProperty('occupationLabel') ? item.occupationLabel.value : 'unknown';
           let residence = item.hasOwnProperty('residenceLabel') ? item.residenceLabel.value : '';
           let residenceCoords = item.hasOwnProperty('residenceCoords') ? this.convertPointToLongLatArray(item.residenceCoords.value) : '';
           let socialClassification = item.hasOwnProperty('socialClassificationLabel') ? item.socialClassificationLabel.value : 'unknown';
           let placeOfDeath = item.hasOwnProperty('placeOfDeathLabel') ? item.placeOfDeathLabel.value : '';
           let placeOfDeathCoords = item.hasOwnProperty('placeOfDeathCoords') ? this.convertPointToLongLatArray(item.placeOfDeathCoords.value) : '';
           let mannerOfDeath = item.hasOwnProperty('mannerOfDeathLabel') ? item.mannerOfDeathLabel.value : '';
           let detentionLocation = item.hasOwnProperty('detentionLocationLabel') ? item.detentionLocationLabel.value : '';
           let detentionLocationCoords = item.hasOwnProperty('detentionLocationCoords') ? this.convertPointToLongLatArray(item.detentionLocationCoords.value) : '';
           let wikiPage = this.getItemWikiPage(item);
           let investigationDate = item.hasOwnProperty('investigationDate') ? item.investigationDate.value : 'N/A';
           let year = 1650;

           if(investigationDate!=='N/A') {
             year = this.getYearFromWikiDate(investigationDate);
             investigationDate = this.convertWikiDateToFriendlyDate(investigationDate);
           }

           // add to list of occupations
           if(!this.occupations.find(obj => obj.type === occupation)){
             this.occupations.push({type: occupation, active: true, iconUrl: this.icons[this.occupations.length]});
           }

           // add to list of social classes
           if(!this.socials.find(obj => obj.type === socialClassification)){
             this.socials.push({type: socialClassification, active: true, iconUrl: this.icons[this.socials.length]});
           }

           // find if witch has already exists
           let witch = witches.find( witch => {
             return witch.id ===  id;
           });

           // if witch exists we have a duplicate. this witch must have either multiple residence or multiple detentions
           // push
           if(witch){

             if(detentionLocation !== ''){
               if(!witch.detentions.find( obj => obj.location === detentionLocation)) {
                 witch.detentions.push({location: detentionLocation, coords: detentionLocationCoords});
                 continue;
               }
             }

             if(residence !== ''){
               if(!witch.residences.find( obj => obj.location === residence)) {
                 witch.residences.push({location: residence, coords: residenceCoords});
                 this.addWitchToMarkers(witch, residence, residenceCoords);
                 continue;
               }
             }

             if(investigationDate !== 'N/A' && witch.investigationDate === 'N/A') {
               witch.investigationDate = investigationDate;
               witch.year = year;
             }

           } else {

             witch = {
               id: id,
               location: residence,
               name: item.itemLabel.value,
               link: 'http://witches.shca.ed.ac.uk/index.cfm?fuseaction=home.accusedrecord&accusedref=' + item.link.value + '&search_string=lastname',
               longLat: residenceCoords,
               sex: sex,
               occupation: occupation,
               socialClassification: socialClassification,
               wikiPage: wikiPage,
               hasWikiPage: wikiPage === '' ? 'no wiki' : 'has wiki',
               residences: [],
               placeOfDeath: placeOfDeath,
               placeOfDeathCoords: placeOfDeathCoords,
               mannerOfDeath: mannerOfDeath,
               detentions: [],
               investigationDate: investigationDate,
               year: year,
             }

             if(residence !== ''){
               witch.residences.push({location: residence, coords : residenceCoords});
             }

             if(detentionLocation !== ''){
               witch.detentions.push({location: detentionLocation, coords : detentionLocationCoords});
             }

             witches.push(witch);
             this.addWitchToMarkers(witch, residence, residenceCoords);
           }

         }

         this.noItems = witches.length;
         this.originalMarkers = JSON.parse(JSON.stringify(this.markers));
         this.saveDataToLocalStorage();
         this.loading = false;
       });


     },
     addWitchToMarkers: function( witch, location, locationCoords ){
       // find marker for current location so you can add witch
       let marker = this.markers.find( marker => {
         return marker.location === location;
       });

       // if a marker exists for the witche's location add the witch to it. if not create a new marker for the location and add the witch.
       if(marker){
         marker.witches.push(witch);
       } else {
         let marker = {
           location: location,
           longLat: locationCoords,
           witches: [witch],
         }

         this.markers.push(marker);
       }
     },
     filterMarkers : function(){
       let markers = JSON.parse(JSON.stringify(this.originalMarkers));
       let layerCollection = this[this.currentLayer.id];
       for (let i = 0; i < layerCollection.length; i++) {
         if (layerCollection[i].active === false) {
           markers.forEach(marker => {
             marker.witches = marker.witches.filter(witch => layerCollection[i].type !== witch[this.currentLayer.property]);
           });
         }
       }

       markers.forEach(marker => {
         marker.witches = marker.witches.filter(witch => witch.year >= this.sliderYear[0] && witch.year <= this.sliderYear[1]);
       });

       this.markers = markers;
     },
     hasWikiEntry : function( marker ){
       let witchesWithEntry = marker.witches.filter( witch => witch.wikiPage !== '');
       return witchesWithEntry.length > 0;
     },
     getItemWikiPage : function( item ){
       let wikiPage = '';

       for(let i = 0; i < this.wikiPages.length; i++){
         if(this.wikiPages[i].id === item.item.value){
           wikiPage = this.wikiPages[i].pageTitle;
           wikiPage.split(' ').join('_');
           wikiPage = 'https://en.wikipedia.org/wiki/' + wikiPage;
         }
       }

       return wikiPage;
     },
     getIcon : function( marker ) {
       let layerCollection = this[this.currentLayer.id];
       let type = this.getMarkerType( marker, layerCollection, this.currentLayer.property );
       let iconUrl = '';

       if( type === 'mixed' ) {
         iconUrl = '/images/witch-single-purple.png';
       } else {
         let item = layerCollection.find( item => item.type === type );
         iconUrl = item.iconUrl;
       }

       return iconUrl;
     },
     getMarkerType : function( marker, layerCollection, property) {
       let type = '';
       let noDifferntTypes = 0;
       for(let i = 0; i < layerCollection.length; i++){
         let witches = marker.witches.filter( witch => {
           return witch[property] === layerCollection[i].type;
         });

         if(witches.length > 0){
           type = layerCollection[i].type;
           noDifferntTypes++;
         }

         if(noDifferntTypes > 1){
           return 'mixed';
         }
       }

       return type;
     },
     filterTiles : function( tile ){
       this.currentTileName = tile.name;
       this.url = tile.url;
     },
     filterLayers : function( layer ){
       this.currentLayer = layer;
       this.filterMarkers();
     },
     flyTo : function( coords ){
       this.$refs.myMap.mapObject.flyTo(coords ,14);
     },
     hasLocalStorageExpired : function(){
       let hours = 24; // Reset when storage is more than 24hours
       let now = new Date().getTime();
       let setupTime = localStorage.getItem('timeline_setupTime');
       return setupTime === null || (now - setupTime > hours*60*60*1000);
     },
     loadDataFromLocalStorage : function(){
       this.occupations = JSON.parse(localStorage.getItem('timeline_occupations'));
       this.socials = JSON.parse(localStorage.getItem('timeline_socials'));
       this.markers = JSON.parse(localStorage.getItem('timeline_markers'));
       this.originalMarkers = JSON.parse(JSON.stringify(this.markers));
       this.noItems = localStorage.getItem('timeline_noItems');
     },
     saveDataToLocalStorage : function(){
       let now = new Date().getTime();
       localStorage.setItem('timeline_setupTime', now);
       localStorage.setItem('timeline_markers', JSON.stringify(this.markers));
       localStorage.setItem('timeline_noItems', this.noItems);
       localStorage.setItem('timeline_occupations', JSON.stringify(this.occupations));
       localStorage.setItem('timeline_socials', JSON.stringify(this.socials));
     },
     toggleFilters : function() {
       this.filters = ! this.filters;
     },
     showPageInfo(){
       this.$swal({
         title: 'Places of Residence for Accused Witches (total named accused witches: 3141)',
         html: '<div>This map shows the geographical residence location for each accused witch in Scotland taken from the Survey of Scottish Witchcraft Database. Out of the <b class="font-bold">3212</b> accused witches whose names are known, the residence for <b class="font-bold">3141</b> witches has been located. The majority of the residences are accurately located down to the precise settlement, while others range from parish to county depending on the records surviving for each accused witch. There is a total of 821 different locations recorded in the database; all but 25 of these have been identified. The remaining unidentified place-names are currently recorded as \‘County of’\ on the map.</div>',

         footer: 'witches.is.ed.ac.uk',
         confirmButtonText: 'Close',
         type: 'info',
         showCloseButton: true,
       });
     },
     convertWikiDateToFriendlyDate(wikiDate){
       let dateYear = wikiDate.substr(0, 4);
       let dateMonth = wikiDate.substr(5, 2);
       let dateDay = wikiDate.substr(8, 2);
       return dateDay + '/' + dateMonth + '/' + dateYear;
     },
     getYearFromWikiDate(wikiDate){
       return wikiDate.substr(0, 4);
     },
     filterDates : function(){
       let markers = JSON.parse(JSON.stringify(this.originalMarkers));

       markers.forEach(marker => {
         marker.witches = marker.witches.filter(witch => witch.year >= this.sliderYear[0] && witch.year <= this.sliderYear[1]);
       });

       this.markers = markers;
     },
   },
   computed : {
     activeMarkers : function() {
       return this.markers.filter(function(marker) {
         return marker.witches.length > 0;
       });
     },
     iconAnchor : function() {
       return [11, 41];
     },
     shadowUrl : function() {
       return '/images/North-Berwick-witch-shadow.png';
     }
   },
   mounted: function() {
     if(this.hasLocalStorageExpired()){
       localStorage.clear();
       this.loadWikiEntries();
     } else {
       this.loadDataFromLocalStorage();
       this.loading = false;
     }
   }
 };
</script>

<style>
 .zoomed-in-img {
   float: left;
   width: 25px;
   height: 38px;
 }

 .icon-shadow{
   position: absolute;
   top: 15px !important;
   left: 0;
   z-index: -1;
   width: 32px;
   height: 22px !important;
 }

</style>