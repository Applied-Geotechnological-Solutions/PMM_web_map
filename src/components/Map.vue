<template>
<img src="https://s3.eu-west-2.amazonaws.com/pleo-polska-misja-medyczna/Logo/home_logo_pmm_wersja-podstawowa_granat-i-czerwony_pl_v2.svg" class='header' style="margin-left: 10vh; margin-top:2vh; margin-bottom:2vh; ">
  

  <div class="grid-container">
        
        <div class="left">
            <div class="toc">

                <h3 class="header-orange" > LAYERS </h3>
                <hr style="margin-left: 0%; margin-right: 25%; height: 3px; background: white;">
                
                <div v-for="item in legend_items" :key="item" class="overlay-text-feed">

                  <input type="checkbox" :id="item.visible" :value="item.visible" @change="change_layer_visibility(item.name)" v-model="item.visible">
                  
                  <label :for="item.name">{{item.name.replace("_", " ")}}</label> <br>
                  
                
                </div>


                

              <div id="CF" ref="yourDiv"></div>
                
            </div>

        </div>

      <div class="right">

        <div id="map" class="map__x" ref="mapCom"></div>

        <button @click='toggle = !toggle' ref="legend__toggle_overlay" class="overlay-legend-toggle"> Legend </button>

        <div v-show='toggle' ref="legend_overlay" class="overlay-legend">

          <h3 style="margin: 0px; padding: 0px; color:white; ">Legend</h3>

                <div v-for="item in legend_items" :key="item" class="legend-list">

                  <div v-if="item.visible & item.inlegend" class="text-bold-white">
                    <img :src="item.img" class="legend-img"> {{item.name.replace("_", " ")}} 
                  </div>

                </div>

        </div>

        <div ref="popupCom2" class="popup2"> Symbology based on UN OCHA resource sharing standard 

          <span class="icon-close" @click="closePopup">x</span>
        </div>


          <div ref="popupCom" class="popup">

              <span class="icon-close" @click="closePopup"> x </span>
              
              <div class="header-orange" > Details: </div> 
              <hr>
              
                <div class="overlay-text-info" v-for="item in popup_content_info" :key="item"> 
                  
                  <div v-html="item"> </div>
                  
                </div>
                <hr style="height: 0.1px; background:#fff;"> 


              <div class="overlay-text-feed" v-html="popup_content_feed"> </div>

          </div>

        </div>

    </div>

</template>


<script setup> 

import { ref, onMounted} from 'vue'
import { Map, View } from 'ol' // Introduce container binding module and view module  
import XYZ from 'ol/source/XYZ' // introduce XYZ Map format  
import Overlay from 'ol/Overlay'// Introduce the cover module  
import 'ol/ol.css' // ol Provided css style （ Must introduce ）

import TileLayer from 'ol/layer/Tile'
//import TileWMS from 'ol/source/TileWMS';
import VectorLayer from 'ol/layer/Vector'
import VectorSource from 'ol/source/Vector'
import GeoJSON from 'ol/format/GeoJSON'
import {Icon, Fill, Stroke, Style} from 'ol/style';
import OSM from 'ol/source/OSM'
//import Select from 'ol/interaction/Select';
//import {click} from 'ol/events/condition';
import {OverviewMap, defaults as defaultControls, ZoomSlider} from 'ol/control';
import MousePosition from 'ol/control/MousePosition';
import {createStringXY} from 'ol/coordinate';
import ScaleLine from 'ol/control/ScaleLine';



//import questionIcon from "@/assets/questionIcon.png";



const map = ref(null) // Examples of maps 
//const icons_link = ref('https://github.com/Syverpet/prototyping_data_april_2022/tree/main/icons/') // Github link to icons folder


// WHENNEWOVERLAY
const mapCom = ref(null) // Map container  
const popupCom = ref(null) // Pop up containerlocal_host
const popupCom2 = ref(null) // Pop up container
const legend__toggle_overlay = ref(null) // Pop up container  
const legend_overlay = ref(null) // Pop up container  
 // WHENNEWOVERLAY
const overlay = ref(null) 
const overlay2 = ref(null)
const overlay3 = ref(null) 
const overlay4 = ref(null) 
const currentCoordinate = ref('') // Pop up information

const popup_content_info = ref('')

const popup_content_feed = ref('')


const keys_list = ref([])


const layers = ref(null)

const viewResolution = ref(null)



const selected = ref(null)

const selectStyle = new Style({
  fill: new Fill({
    color: '#eeeeee',
  }),
  stroke: new Stroke({
    color: 'rgba(255, 255, 255, 0.7)',
    width: 2,
  }),
});


// Legend list items // WHENADDLAYERS
const legend_items = ref([
   {
    id: 0,
    name: 'Google_Satellite',
    img: 'https://github.com//Syverpet/prototyping_data_april_2022/blob/main/icons/Medical_Equipment_and_Supplies.png?raw=true',
    visible: true,
    inlegend: false
  },
  {
    id: 0,
    name: 'OSM_baselayer',
    img: 'https://github.com//Syverpet/prototyping_data_april_2022/blob/main/icons/Medical_Equipment_and_Supplies.png?raw=true',
    visible: true,
    inlegend: false
  },
  { 
    id: 1,
    name: 'Dark_baselayer',
    img: 'https://github.com//Syverpet/prototyping_data_april_2022/blob/main/icons/Medical Facilities.png?raw=true',
    visible: true,
    inlegend: false
  },
  {
    id: 2,
    name: 'Hospital_Support',
    img: 'https://github.com//Syverpet/prototyping_data_april_2022/blob/main/icons/Medical%20Facilities.png?raw=true',
    visible: true,
    inlegend: true
  }
])



//BASE LAYERS // WHENADDLAYERS

const Google_Satellite = ref(

    new TileLayer({
    name: 'Google_Satellite',
    visible: legend_items.value[1].visible,
    source: new XYZ({
          url: 'http://mt0.google.com/vt/lyrs=s&hl=en&x={x}&y={y}&z={z}'
      }),
    })
  )

const OSM_baselayer = ref(new TileLayer({
  name: 'OSM_baselayer',
  visible: legend_items.value[1].visible,
  source: new OSM(), // tiles are served by OpenStreetMap
  view: new View({ 
    projection: 'EPSG:4326', // Projection coordinate system  
    center: [113.1206, 23.034996], // The center of the map  
    zoom: 1
  })
}))

const Dark_baselayer = ref(new TileLayer({
  name: 'Dark_baselayer',
  visible: legend_items.value[2].visible,
  source: new XYZ({
    url: "https://c.tile.jawg.io/jawg-dark/{z}/{x}/{y}.png?access-token=87PWIbRaZAGNmYDjlYsLkeTVJpQeCfl2Y61mcHopxXqSdxXExoTLEv7dwqBwSWuJ",
    view: new View({ projection: 'EPSG:4326', // Projection coordinate system  
    center: [113.1206, 23.034996], // The center of the map  
    zoom: 1, // Map zoom level （ The default level when opening a page,
    }),
  })
}))


// VECTOR LAYERS 

const Hospital_Support = ref(new VectorLayer({
      name: 'Hospital_Support',
      visible: legend_items.value[3].visible,
      source: new VectorSource({
        format: new GeoJSON(),
        url: 'https://raw.githubusercontent.com/Syverpet/prototyping_data_april_2022/main/geojson_test.json.geojson'
      }),
      style: new Style({
        image: new Icon({
          src: legend_items.value[3].img,
        }),
      })
  }))




// MAP CONTROLS

const overviewMapControl = new OverviewMap({
  //className : 'overviewMap',
  layers: [
    new TileLayer({
      source: new OSM(),
    }),
  ],
});

const mousePositionControl = new MousePosition({
  className: 'mouse-position',
  coordinateFormat: createStringXY(4),
  projection: 'EPSG:4326',
  // comment the following two lines to have the mouse position
  // be placed within the map.
});

const zoomslider = new ZoomSlider({

      //className : 'zoomslider'
    }
  
  );

const scaleLine = new ScaleLine({
    //className : 'scaleLine'
  }
);

const toggle = ref(false)




// Initialize map  
function initMap() {  
  //OVERLAY  // WHENNEWOVERLAY
  overlay.value = new Overlay({ element: popupCom.value, 
  autoPan: true, 
  autoPanAnimation: { 
  duration: 250 } })

  overlay2.value = new Overlay({ element: popupCom2.value, 
  autoPan: true,
  position: [3515315.442215883, 6034799.1294523671],
  autoPanAnimation: { 
  duration: 250 } })

  overlay3.value = new Overlay({ element: legend_overlay.value, 
  autoPan: true,
  position: [],
  autoPanAnimation: {
  duration: 250 } })

  overlay4.value = new Overlay({ element: legend__toggle_overlay.value, 
  autoPan: true,
  position: [],
  autoPanAnimation: {
  duration: 250 } })


  
  //MAP
  map.value = new Map({ 
    target: mapCom.value,
    controls: defaultControls().extend([overviewMapControl, mousePositionControl, zoomslider, scaleLine]),
    layers: [ // WHENADDLAYER
      Dark_baselayer.value,
      OSM_baselayer.value,
      Google_Satellite.value,
      Hospital_Support.value
     ], 
      view: new View({ projection: 'EPSG:3857', // Projection coordinate system  
      center: [3515315.442215883, 6034799.1294523671], // The center of the map  
      zoom: 5 // Map zoom level （ The default level when opening a page ） 
      }),
      overlays: [overlay.value, overlay2.value, overlay3.value, overlay4.value] // WHENNEWOVERLAY
       // Bind a cover
      })

  map.value.controls.getArray()[0].getProperties()

  mapClick() // Bind the click event after map initialization  
  
  
  
} // Click map event  

function closePopup() { 
    overlay.value.setPosition(undefined)
    overlay2.value.setPosition(undefined) // setPosition Pass in undefined Will hide pop-up elements  
    currentCoordinate.value = '' // Empty the pop-up window  
}




function change_layer_visibility(layer_name) {
  console.log(layer_name)
  map.value.getLayers().forEach(function(layer) {
  if (layer.get('name') === layer_name) {
    layer.setVisible(!layer.getVisible())
  }
})

  
  //dark_base.setVisible(!dark_base.getVisible())

}



function mapClick() { 


    map.value.on('singleclick', event => { // Bind a click event'

   
    
    //console.log('target', map.value)

    var startTime = performance.now()                                            // TIMETIMETIMETIME START

      const coordinate = event.coordinate // Get coordinates  
      currentCoordinate.value = coordinate // Save coordinate points  
       // Set where the cover appears

      


      viewResolution.value = map.value.getView().getResolution()

      layers.value = map.value.getLayers().getArray();

      console.log(coordinate)
      





      //for(var i in layers.value){

          //console.log('i', i)

          //console.log('Properties', this.layers[i].getProperties())
          //console.log('Source', this.layers[i].getSource().serverType_)

          //if (layers.value[i].getSource().serverType_ == "geoserver") {

            //console.log('TILE layer filtered:', layers.value[i])

           

          
          if (selected.value !== null) { 
                    selected.value.setStyle(undefined);
                    selected.value = null;
          }
              

              map.value.forEachFeatureAtPixel(event.pixel, function (feature) { // can add layer as argument **
 
                  console.log('FEATURE', feature.values_)

                  selected.value = feature;
                  selectStyle.getFill().setColor(feature.get('COLOR') || '#eeeeee');
                  feature.setStyle(selectStyle);
                  return true;
              });

          if (selected.value) { 

            keys_list.value = [] // reset list

            console.log(keys_list)

                var keys = selected.value.getKeys() // keys

                for (var n in keys) {

                    var key = keys[n]
                    
                    keys_list.value.push(key)
                }

               var list_of_remove_keys = ['geometry', 'Shape_Leng', 'Shape_Area', 'Area_Km', 'Lat', 'Long', 'coordinates', 'attributes'] //   REMOVE KEYS HERE! 

               var keys_list2 = keys_list

               
                  
                for (var remove_key in list_of_remove_keys) { // loop for remove keys

                  var k3 = list_of_remove_keys[remove_key]

                  var position = keys_list2.value.indexOf(k3) // get index of remove key

                  //console.log('removeKey', k3)

                  //console.log('keylist_length', keys_list2.value.length)

                  //console.log('position', position)

                  //console.log('splice', keys_list.value.splice(position, 1))

                  if (position != -1) {keys_list2.value.splice(position, 1) 
 
                  }
                }
                

                // Create lists of pupup content strings

                popup_content_info.value = []
                popup_content_feed.value = '<h3 class="header-orange">' + 'Support given:\n'.bold() + '</h3>' + selected.value.get('attributes').replace(/£/g, ".\n\n<hr style='margin-left: 45%; margin-right: 45%; border: 1; '> \n")

                for (var k1 in keys_list2.value) { // keys list filter keys and push to popup content

                    

                    var k2 = keys_list2.value[k1]

                    //console.log('key', k2)

                    popup_content_info.value.push(k2.bold() + ':\n ' + selected.value.get(k2))  // push both key and value to final content list

                    
                }

                  

                  overlay.value.setPosition(event.coordinate) // set coordinate of overlay for activation of popup

          }

                
          else {   // If there is no feature in pixel
            popup_content_info.value = null;
            overlay.value.setPosition(null)
          }

          var endTime = performance.now()                                              // TIMETIMETIMETIME END
          console.log(`Map click took ${endTime - startTime} milliseconds`)
      
    })
  } // Close the pop-up window  

//watch( 

//zoomChanged(currentZoom) {
//      this.currentZoom = currentZoom;
//    },
//)

//onMounted(() => { watch
  
//  console.log(legend_items)
//})

onMounted(() => {
     // Perform map initialization after the element is loaded  
  initMap()
  })
  

  </script>


<style > 

.map__x { 
  width: 100%;  
  height: 100%; 
  border: 1px solid #eee; 
  }



.mouse-position {
  color: rgb(255, 65, 36);
  font-size: 15px;
  font-family: Roboto, Arial, sans-serif;
  padding-left: 650px
}

.ol-overviewmap {
  bottom: 50px;
  

}


 
  
/*  .zoomslider {
       padding: 0px;
        
      }
.overviewMap {
    left: 0.5em;
    bottom: 0.5em;
}

*/

.popup { 
  background-color: rgb(123, 152, 188, 0.7);
  width: 25vh;
  height: 10vh;
  color: white;
  box-shadow: 0 5px 10px rgb(2 2 2 / 40%);
  
  padding: 28px 25px;
  font-size: 13px;
  overflow:auto;
  transition: all 0.3s ease-in-out;
  
  border-radius: 7.5px;
  }
  .popup:hover {
  background-color: rgb(23, 9, 46, 0.9);;
  height: 30vh;
}

.popup2 {
  margin: 70px auto;
  padding: 20px;
  
  background-color: rgb(123, 152, 188);
  border-radius: 5px;
  box-shadow: 0 5px 10px rgb(2 2 2 / 40%);
  width: 30%;
  position: relative;
  transition: all 0.3s ease-in-out;
  color: #333;
  font-family: Tahoma, Arial, sans-serif;
}
.popup2:hover {
  background-color: rgba(0,60,136,0.7);
}

.overlay-legend {
display: block;
min-height: 100px;
overflow: hidden;

margin: 40px auto;
padding: 15px;
border: 1px solid rgba(0,110,172,1);
box-shadow: 0 5px 10px rgb(2 2 2 / 40%);
border-radius: 2.5px;

width: 16vh;

background-color: rgb(123, 152, 188);
font-family: Tahoma, Arial, sans-serif;

transition: all 0.5s ease-in-out;
position: absolute;
left: 60px;
top: 20px;
}
.overlay-legend:hover {
  background-color: rgba(0,60,136,0.7);
}

.overlay-legend-toggle {
background-color: rgb(123, 152, 188);
height: 25px;
width: 80px;
color: #fff;
border: 0.2px;
border-radius: 1.5px;
border-color: rgba(255, 255, 255, 0.849);
position: absolute;
left: 60px;
top: 9px;
}
.overlay-legend-toggle:hover {
  background-color: rgba(0,60,136,0.7);
  
}

.icon-close { 
  position: absolute; 
  top: 13px; 
  right: 16px;
  cursor: pointer;
  color: rgb(255, 255, 255);
  font-size: 15px;
font-family: Silka, sans-serif;
  border: 2px;
  border-color: rgba(0, 0, 0, 0.849);
  } 

.overlay-text-header {
color: white;
font-size: 16px;
font-family: Silka, sans-serif;
font-weight: bold;
}

.overlay-text-feed {
color: white;
font-size: 13px;
font-family: Silka, sans-serif;
white-space: pre-wrap;
line-height: 1.5;
}

.overlay-text-info {
color: rgb(255, 255, 255);
font-size: 13px;
font-family: Silka, sans-serif;
white-space: pre-wrap;
line-height: 1.5;
position:relative;
padding-bottom: 5px;
stroke-width: 3px;
-webkit-text-stroke-color: white;

}

.header {
    grid-area: header;
    font-size: 25px;
    height: 5vh;
    font-family: 'Gill Sans','Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
}



.grid-container {
    display: grid;
    grid-template-areas: 
    'header header'
    'left right';
}

.left {
    grid-area: left;
    width: 10vw;
    height: 90vh;
    background: rgb(23, 9, 46);
    border-radius: 0.5vh;
}

.right {
    grid-area: right;
    width: 90vw;
    height: 90vh;
    border: 0px solid rgb(0, 0, 0);
    border-radius: 1.5vh;
}

.toc {
    margin-left: 10px;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}




.legend-img {
  position: relative;
  top: 1.2em;
  padding: 0px;
  margin: 0px;
  height: 40px;
}

.legend-list {
  margin-block-start: 0em;
  margin-block-end: 0em;
  margin-inline-start: 0px;
  margin-inline-end: 0px;
  /*padding-inline-start: 35px */
}

.legend-text {
  padding: 0px;
  margin: 0px;
}

/* GENERAL CLASES */

.text-bold-white {
color: rgb(255, 255, 255);
font-size: 13px;
position:relative;
padding-bottom: 5px;
bottom: 5px;
font-family: Silka, sans-serif;
font-weight: bold;
}

.text-bold-black {
color: rgb(0, 0, 0);
font-size: 13px;
font-family: Silka, sans-serif;
font-weight: bold;
}

.text-bold-purple {
color: rgb(23, 9, 46);
font-size: 13px;
font-family: Silka, sans-serif;
font-weight: bold;
}

.text-bold-orange {
color: rgb(255, 65, 36);
font-size: 13px;
font-family: Silka, sans-serif;
font-weight: bold;
}

.header-orange {
color: rgb(255, 65, 36);
font-size: 16px;
font-family: Silka, sans-serif;
font-weight: bold;
font-weight: 700;
font-family: Silka, sans-serif;
}




::-webkit-scrollbar {
  
  background-color: rgba(255,255,255,0.4);
  width: 11px;
  border-radius: 10px;
  padding: 10px;
  cursor:pointer;
}

::-webkit-scrollbar-thumb {
  background-color: rgb(123, 152, 188, 0.7);
  border-radius: 10px;
  border: 3px rgba(255,255,255,0.4);
}

::-webkit-scrollbar-track {
    background-color: rgba(255,255,255,0.4);
}

::-webkit-scrollbar-thumb:hover {
  background-color: rgb(123, 152, 188);
}

.hr {
  border: 0;
  border-top: 1px solid #CCC;
}

.hr2 {
  border: 0;
  border-top: 1px dashed #CCC;
}


/* NOTES 

background-color: rgba(255,255,255,0.4);  Grey background ol default

background-color: rgba(0,60,136,0.5); Blue background ol default

border-radius: 2px 2px 0 0;

box-shadow: 0 5px 10px rgb(173, 173, 173);

rba(255, 65, 36) #orange
rba(23, 9, 46) #purpule

font-size: 15px;

font-weight: 700;

*/


</style>