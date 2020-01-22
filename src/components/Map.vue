<template lang="pug">
#map.absolute.absolute--fill
</template>

<script>
import mapboxgl from 'mapbox-gl';
import U from 'mapbox-gl-utils';


const sources = {
    neighborhoods: {
        mapbox: 'mapbox://sbcherre.neighborhoods',
    },
    // parcels: {
    //     mapbox: 'mapbox://sbcherre.parcelssmall'
    // },
    saz: {
        mapbox: 'mapbox://sbcherre.saz'
    }
}

function getSource(layer) {
    if (window.location.hostname !== 'localhost' || window.location.hash.match(/mapbox/)) {
        return sources[layer] ? sources[layer].mapbox : `mapbox://sbcherre.${layer}`;
    } else {
        // return { tiles: [`http://localhost:4004/${layer}/{z}/{x}/{y}.pbf`] }
        return `http://localhost:4004/${layer}/index.json`
    }
}

export default {
    async mounted() {
        mapboxgl.accessToken = window.location.hash.match(/token=(pk\.[-._a-zA-Z0-9]+)/)[1];
        const map = new mapboxgl.Map({
            container: 'map',
            center: [-74.6631, 40.7863],
            zoom: 5,
            style: 'mapbox://styles/mapbox/light-v9',
            hash: 'loc'
        });
        U.init(map, mapboxgl);
        window.map = map;
        window.app.Map = this;

        const visible = layer =>  ({
            // census: true,
            // zip: true,
            // places: true,
            // county: true,
            // state: true,
            parcels:true
        }[layer] ? 'visible' : 'none');


        map.U.onLoad(() => {
            // map.U.addVector('neighborhoods', 'http://localhost:3050/{z}/{x}/{y}.pbf');
            map.U.addVector('neighborhoods', getSource('neighborhoods'));
            map.U.addLine('neighborhoodsmt-line', 'neighborhoods', {
                sourceLayer: 'neighborhoods-mt',
                lineWidth: ['interpolate', ['linear'], ['zoom'], 
                    5, 0.1,
                    7, 1
                ],
                lineColor: 'hsl(330, 80%, 40%)',
                visibility: visible('neighborhoods')
            });
            map.U.addFill('neighborhoodsmt-fill', 'neighborhoods', {
                sourceLayer: 'neighborhoods-mt',
                fillColor: 'hsla(330, 80%, 40%,0.1)',
                visibility: visible('neighborhoods')

                
            });
            map.U.addLine('neighborhoodsrs-line', 'neighborhoods', {
                sourceLayer: 'neighborhoods-rs',
                lineWidth: ['interpolate', ['linear'], ['zoom'], 
                    5, ['case', ['to-boolean', ['feature-state','hover']], 2, 0.1],
                    7, ['case', ['to-boolean', ['feature-state','hover']], 4, 1],
                ],
                // lineColor: 'hsl(230, 80%, 40%)'
                lineColor: ['case', ['to-boolean', ['feature-state','hover']], 'hsl(230, 100%, 30%)', 'hsl(230, 80%, 40%)'],
                visibility: visible('neighborhoods')


            });
            map.U.addFill('neighborhoodsrs-fill', 'neighborhoods', {
                sourceLayer: 'neighborhoods-rs',
                fillColor: 'hsla(230, 80%, 40%,0.1)',
                // fillColor: ['case', ['to-boolean', ['feature-state','hover']], 'hsl(230, 80%, 0%)', 'hsla(230, 80%, 40%,0.1)']
                visibility: visible('neighborhoods')
                
            });

            map.U.addSymbol('neighborhoodsrs-labels', 'neighborhoods', {
                sourceLayer: 'neighborhoods-rs',
                textField: '{GEO_NAME}',
                textColor: 'hsla(230, 80%, 40%,0.8)',
                textSize: 12,
                visibility: visible('neighborhoods')

            });


            map.U.addLine('neighborhoodsnd-line', 'neighborhoods', {
                sourceLayer: 'neighborhoods-nd',
                lineWidth: ['interpolate', ['linear'], ['zoom'], 
                    5, 0.1,
                    7, 1
                ],
                lineColor: ['case', ['to-boolean', ['get','hover']], 'hsl(130, 80%, 20%)', 'hsl(130, 80%, 40%)'],
                visibility: visible('neighborhoods')
            });
            map.U.addFill('neighborhoodsnd-fill', 'neighborhoods', {
                sourceLayer: 'neighborhoods-nd',
                fillColor: 'hsla(130, 80%, 40%,0.1)',
                visibility: visible('neighborhoods'),
                
            });

            map.U.addSymbol('neighborhoodsnd-labels', 'neighborhoods', {
                sourceLayer: 'neighborhoods-nd',
                textField: '{GEO_NAME}',
                textColor: 'hsla(130, 80%, 40%,0.8)',
                textSize: 16,
                visibility: visible('neighborhoods'),
            });

            map.U.addVector('parcels', getSource('parcels'))
                .addLine('parcels-line', {
                    sourceLayer: 'parcels',
                    lineColor: 'cyan',
                    visibility: visible('parcels'),
                })
                .addFill('parcels-fill', {
                    sourceLayer: 'parcels',
                    visibility: visible('parcels'),
                    // fillColor: 'transparent'
                    fillColor: ['interpolate-hcl', ['linear'], ['%', ['to-number', ['get', 'CountyFIPS']], 20],
                        0, 'hsla(0, 80%,50%,0.5)',
                        9, 'hsla(240,80%,50%,0.5)',

                    ]
                });


            map.U.addVector('saz', getSource('saz'))
                .addLine('saz-line', {
                    sourceLayer: 'saz',
                    lineColor: 'pink',
                    lineDasharray: [8,4],
                    visibility: visible('saz'),
                });

            map.U.addVector('census', getSource('census'))
                .addFill('zip-fill', { 
                    sourceLayer: 'census-zi',
                    fillColor: ['case', ['to-boolean', ['feature-state','hover']], 'hsla(330,80%,80%,0.5)', 'transparent'],
                    visibility: visible('zip'),
                })
                .addLine('zip-line', {
                    sourceLayer: 'census-zi',
                    lineWidth: 1,
                    visibility: visible('zip'),
                })
                .addLine('county-line', {
                    sourceLayer: 'census-co',
                    lineColor: 'hsl(90,80%,50%)',
                    lineWidth: 1,
                    visibility: visible('county'),
                })
                .addLine('state-line', {
                    sourceLayer: 'census-st',
                    lineColor: 'hsl(140,80%,50%)',
                    lineWidth: 1,
                    visibility: visible('state'),
                })
                .addLine('places-line', {
                    sourceLayer: 'census-pl',
                    lineColor: 'hsl(240,80%,50%)',
                    lineWidth: 1,
                    visibility: visible('places'),
                })

            
            
        }); 
        const fillLayers = ['zip-fill', 'neighborhoodsrs-fill','neighborhoodsnd-fill', 'neighborhoodsmt-fill', 'parcels-fill'];
        map.U.hoverPointer(fillLayers);
        map.U.hoverFeatureState('neighborhoodsrs-fill', 'neighborhoods', 'neighborhoodsrsmz', e => {
            console.log(e.features[0].state)
            // window.app.FeatureInfo.feature = e.features[0];
        });
        // map.U.hoverFeatureState('zip-fill', 'zip', 'zip', e => {
        //     // console.log(e.features[0].state)
        //     // window.app.FeatureInfo.feature = e.features[0];
        // });
        map.U.clickOneLayer(fillLayers, e => {
            const f = e.features[0];
            console.log(e.features[0].sourceLayer);
            map.setFeatureState({
                id: f.id,
                source: 'neighborhoods',
                sourceLayer: 'db'
            }, { hover: true });

            window.app.FeatureInfo.feature = e.features[0];
        });
        
    }
}
import 'mapbox-gl/dist/mapbox-gl.css';

</script>

<style scoped>
</style>