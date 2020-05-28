<html>
<head>    
<meta charset="utf-8" />
<title>Cluster analysis</title>
<meta name="viewport" content="initial-scale=1,maximum-scale=1,user-scalable=no" />
<script src="https://api.mapbox.com/mapbox-gl-js/v1.10.1/mapbox-gl.js"></script>
<script src="https://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
<script src="https://code.highcharts.com/highcharts.js"></script>
<link href="https://api.mapbox.com/mapbox-gl-js/v1.10.1/mapbox-gl.css" rel="stylesheet" />
<style>
	body { margin: 0; padding: 0; }
	
<html>
<head>    
<meta charset="utf-8" />
<title>Create and style clusters</title>
<meta name="viewport" content="initial-scale=1,maximum-scale=1,user-scalable=no" />
<script src="https://api.mapbox.com/mapbox-gl-js/v1.10.1/mapbox-gl.js"></script>
<script src="https://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
<script src="https://code.highcharts.com/highcharts.js"></script>
<link href="https://api.mapbox.com/mapbox-gl-js/v1.10.1/mapbox-gl.css" rel="stylesheet" />
<style>
	body { margin: 0; padding: 0; }
	
</style>
</head>
<body>
<div id="map" style="position: absolute; top:0; bottom: 0; width: 100%"></div>
<div id="ea" style="position: absolute; top:0px;width: 290px; height: 150px; margin: 5px; opacity:0.8 "></div>
<div id="sa" style=" position: absolute; top: 160px;width: 290px; height:150px; margin: 5px; opacity:0.8 "></div>
<div id="eu" style=" position: absolute; top: 320px; width: 290px; height: 150px; margin: 5px; opacity:0.8 "></div>
<div id="mix" style="position: absolute; top: 480px; width: 290px; height: 150px; margin: 5px; opacity:0.8 "></div>
<div id="show" style="position: absolute;top:0; width: 420px; height: 310px; margin: 5px; opacity:0.8; margin-left: 940px"></div>
<div id="rank" style="position: absolute;top:50%; width:420px; height:310px; margin: 5px; opacity:0.8; margin-left: 940px"></div>
<script>
   
   mapboxgl.accessToken = 'pk.eyJ1IjoiamFuZXhpenp6enoiLCJhIjoiY2s5d3k2eWd1MDlxbDNpcDNhOWVwYm5hOSJ9.-bRRt6ezlyK0YcqlD5epMg';
    //get the map backgrount
    // add the background map
    
    var map = new mapboxgl.Map({
        container: 'map', //call back the map div
        style: 'mapbox://styles/mapbox/light-v10',//get the map style
        center: [-3.192311, 55.94926],//get the center point
        zoom: 12.5 //define the zoom size
    });
    
    let xhr = new XMLHttpRequest(); // call the data (geojson file)
    xhr.open('GET', './resgeo.geojson');
    xhr.responseType = 'json';
    var onError =function (){ }
    xhr.onerror=onError;
    xhr.send();
    xhr.onload = function() {
        if (xhr.status==200){
            //cluster as East Asia
            let res = xhr.response;
            //cluster as south asia
            let sa = xhr.response;
            //middle East and mixcio because the kind is less than 2, mix them up
            let mix =xhr.response;
            //cluster as eu
            let eu = xhr.response;
            // This is only for the classification of restaurants with a country type. For example, a seafood restaurant that does not indicate the country is not counted. 
            for(var  i=res.features.length-1; i>0;i--){
                var obj = res.features[i].properties;
                
                // Thailand is strictly a Southeast Asian restaurant, but for the convenience of classification, it is classified as an East Asian restaurant.
                
                var asia = ["Chinese Restaurant","Thai Restaurant","Japanese Restaurant","Korean Restaurant"];// get the element we want
             
                if (!asia.includes(obj['qualifier_data'])){ // match the data from the data source
                    res.features.splice(i,1); // remove the extraneous variable
                }
            }
            console.log(res) // check the data 
            
             for(var  i=sa.features.length-1; i>0;i--){
                var South_Asia = ['Indian Restaurant','Nepalese Restaurant','Turkish Restaurant'];
                 var obj = sa.features[i].properties;
                
                 if (!South_Asia.includes(obj['qualifier_data'])  ){
                    sa.features.splice(i,1);
                }
            }console.log(sa) 
            
            
            for(var  i=mix.features.length-1; i>0;i--){
                var amer =['Mexican Restaurant','Lebanese Restaurant'];
                 var obj = mix.features[i].properties;
                
                 if (!amer.includes(obj['qualifier_data'])  ){
                    mix.features.splice(i,1);
                }
            }console.log(mix)
            
            for(var  i=eu.features.length-1; i>0;i--){
                // Among them, pizza was invented in Italy, so it was classified as an Italian restaurant. At the same time, because the study place is located in the United Kingdom, the point which does not clearly indicate which country's restaurant classification is defaulted to belong to the EU. on the other hand, The Mediterranean Sea does not belong to any continent. It is the dividing line between Europe and Africa. There is no African-style restaurant here, so it is classified as the EU.
                
                var eup = ['English Restaurant','Greek Restaurant','Mediterranean Restaurant','French Restaurant','Pizza Restaurant','Restaurant'];
                 var obj = eu.features[i].properties;
                
                 if (!eup.includes(obj['qualifier_data'])  ){
                    eu.features.splice(i,1);
                }
            }console.log(eu)
       

   
        map.on('load', function() {
            
        
        // east asian restaurant cluster   
        map.addSource('restaurant', {
            'type': 'geojson',
            'data': res,
            cluster: true,
            clusterMaxZoom: 14, // Max zoom to cluster points on
            clusterRadius: 50 // Radius of each cluster when clustering points (defaults to 50)
        });    
         
       
        map.addLayer({
            id: 'clusters_ea',
            type: 'circle',
            source: 'restaurant',
            filter: ['has', 'point_count'],
            paint: {     
                'circle-color':'#FFFF00',
                'circle-opacity': 0.5,// the poacity of the point


                'circle-radius': [
                    'step',
                    ['get', 'point_count'],//different numbers of restaurants inside the cluster will be shown as different redius.
                    15,
                    10,
                    25,
                    20,
                    35
                    ]
                },
           
            }); 
            
        map.addLayer({
            id: 'cluster-count_ea',
            type: 'symbol',
            source: 'restaurant',
            filter: ['has', 'point_count'],
            layout: {
                'text-field': '{point_count_abbreviated}',
                'text-font': ['DIN Offc Pro Medium', 'Arial Unicode MS Bold'],
                'text-size': 12
            },
                });
            
        // south asian restaurant cluster    
        map.addSource('sa', {
            'type': 'geojson',
            'data': sa,
            cluster: true,
            clusterMaxZoom: 14, // Max zoom to cluster points on
            clusterRadius: 50 // Radius of each cluster when clustering points (defaults to 50)
        });    
         
       
        map.addLayer({
            id: 'clusters_sa',
            type: 'circle',
            source: 'sa',
            filter: ['has', 'point_count'],
            paint: {     
                'circle-color':'#FFB6C1',// get the cluster piont color
                'circle-opacity': 0.5,// the poacity of the point

                'circle-radius': [
                    'step',
                    ['get', 'point_count'],//different numbers of restaurants inside the cluster will be shown as different redius.
                    15,
                    10,
                    25,
                    20,
                    35
                    ]
                },
           
            }); 
            
        map.addLayer({
            id: 'cluster-count_sa',
            type: 'symbol',
            source: 'sa',
            filter: ['has', 'point_count'],
            layout: {
                'text-field': '{point_count_abbreviated}',
                'text-font': ['DIN Offc Pro Medium', 'Arial Unicode MS Bold'],
                'text-size': 12
            },
                });
            
          
       // the cluster of other mixed restaurants  
        map.addSource('mix', {
            'type': 'geojson',
            'data': mix,
            cluster: true,
            clusterMaxZoom: 14, // Max zoom to cluster points on
            clusterRadius: 50 // Radius of each cluster when clustering points (defaults to 50)
        });    
    
        map.addLayer({
            id: 'clusters_mix',
            type: 'circle',
            source: 'mix',
            filter: ['has', 'point_count'],
            paint: {     
                'circle-color':'#7B68EE',
                'circle-opacity': 0.5,// the poacity of the point


                'circle-radius': [
                    'step',
                    ['get', 'point_count'],//different numbers of restaurants inside the cluster will be shown as different redius.
                    15,
                    10,
                    25,
                    20,
                    35
                    ]
                },
           
            }); 
            
        map.addLayer({
            id: 'cluster-count_mix',
            type: 'symbol',
            source: 'mix',
            filter: ['has', 'point_count'],
            layout: {
                'text-field': '{point_count_abbreviated}',
                'text-font': ['DIN Offc Pro Medium', 'Arial Unicode MS Bold'],
                'text-size': 12
            },
                });
            
        // the cluster of eu reataurants    
        map.addSource('eu', {
            'type': 'geojson',
            'data': eu,
            cluster: true,
            clusterMaxZoom: 14, 
            clusterRadius: 50 
        });        
                    
        
        map.addLayer({
            id: 'clusters_eu',
            type: 'circle',
            source: 'eu',
            filter: ['has', 'point_count'],
            paint: {     
                'circle-color':'#7FFFD4',
                'circle-opacity': 0.5,

                'circle-radius': [
                    'step',
                    ['get', 'point_count'],
                    15,
                    10,
                    25,
                    20,
                    35
                    ]
                },
           
            }); 
            
        map.addLayer({
            id: 'cluster-count_eu',
            type: 'symbol',
            source: 'eu',
            filter: ['has', 'point_count'],
            layout: {
                'text-field': '{point_count_abbreviated}',
                'text-font': ['DIN Offc Pro Medium', 'Arial Unicode MS Bold'],
                'text-size': 12
            },
                });
                    

        });
             } else{                   
                    onError();               
                }       
            };
    
    
    //add the pie chart to show the detial, the east asian restaurants
    $(document).ready(function() {  
        var chart = {
            plotBackgroundColor: null,
            plotBorderWidth: null,
            plotShadow: false
        };
        var title = {
            text: 'East Asian restaurants'   
        };      
        var tooltip = {
            pointFormat: '{series.name}: <b>{point.percentage:.1f}%</b>'
        };
        var plotOptions = {
            pie: {
                allowPointSelect: true,
                cursor: 'pointer',
                dataLabels: {
                enabled: true,
                format: '<b>{point.name}</b>: {point.percentage:.1f} %',
                }
            }
        };
        var series= [{
              type: 'pie',
              name: 'ea_type',
              data: [
                 ['Japanese', 0.25],
                 ['Thai', 0.28],
                 {
                    name: 'Chinese',
                    y: 0.38,
                    sliced: true,
                    selected: true
                 },
                 ['Korean', 0.09]
              ]
           }];     

       var json = {};   
       json.chart = chart; 
       json.title = title;     
       json.tooltip = tooltip;  
       json.series = series;
       json.plotOptions = plotOptions;
       $('#ea').highcharts(json);  
    });
    
    
    // the south asian restaurants detials
     $(document).ready(function() {  
        var chart = {
            plotBackgroundColor: null,
            plotBorderWidth: null,
            plotShadow: false
        };
        var title = {
            text: 'South Asian restaurants'   
        };      
        var tooltip = {
            pointFormat: '{series.name}: <b>{point.percentage:.1f}%</b>'
        };
        var plotOptions = {
            pie: {
                allowPointSelect: true,
                cursor: 'pointer',
                dataLabels: {
                enabled: true,
                format: '<b>{point.name}</b>: {point.percentage:.1f} %',
                }
            }
        };
        var series= [{
              type: 'pie',
              name: 'sa_type',
              data: [
                 ['Indian', 0.79],
               
                  ['Nepalese', 0.04],
                {
                    name: 'Turkish',
                    
                    y: 0.17,
                    sliced: true,
                    selected: true
                 },
                
              ]
           }];     

       var json = {};   
       json.chart = chart; 
       json.title = title;     
       json.tooltip = tooltip;  
       json.series = series;
       json.plotOptions = plotOptions;
       $('#sa').highcharts(json);  
    });
    
    // the eu restaurants detials
    
     $(document).ready(function() {  
        var chart = {
            plotBackgroundColor: null,
            plotBorderWidth: null,
            plotShadow: false
        };
        var title = {
            text: 'Europe restaurants'   
        };      
        var tooltip = {
            pointFormat: '{series.name}: <b>{point.percentage:.1f}%</b>'
        };
        var plotOptions = {
            pie: {
                allowPointSelect: true,
                cursor: 'pointer',
                dataLabels: {
                enabled: true,
                format: '<b>{point.name}</b>: {point.percentage:.1f} %',
                }
            }
        };
        var series= [{
              type: 'pie',
              name: 'eu_type',
              data: [
                  ['English', 0.03],
                  ['Greek',0.01],
                  
               
                  ['Italy',0.07],
                  ['French', 0.08],
        
                ['Med', 0.03],
                    {
                    name: 'Others',
                    
                    y: 0.78,
                    sliced: true,
                    selected: true
                 },
                   
              ]
           }];     

       var json = {};   
       json.chart = chart; 
       json.title = title;     
       json.tooltip = tooltip;  
       json.series = series;
       json.plotOptions = plotOptions;
       $('#eu').highcharts(json);  
    });
    
    // other restaurants type
    $(document).ready(function() {  
        var chart = {
            plotBackgroundColor: null,
            plotBorderWidth: null,
            plotShadow: false
        };
        var title = {
            text: 'Other restaurants'   
        };      
        var tooltip = {
            pointFormat: '{series.name}: <b>{point.percentage:.1f}%</b>'
        };
        var plotOptions = {
            pie: {
                allowPointSelect: true,
                cursor: 'pointer',
                dataLabels: {
                enabled: true,
                format: '<b>{point.name}</b>: {point.percentage:.1f} %',
                },
            }
        };
        var series= [{
              type: 'pie',
              name: 'mix_type',
              data: [
                  ['Lebanese', 0.91],
                  ['Mexican',0.09],
              ]
           }];     

       var json = {};   
       json.chart = chart; 
       json.title = title;     
       json.tooltip = tooltip;  
       json.series = series;
       json.plotOptions = plotOptions;
       $('#mix').highcharts(json);  
    });
          
    // the total pie chart
    $(document).ready(function() {  
       var chart = {
           plotBackgroundColor: null,
           plotBorderWidth: null,
           plotShadow: false
       };
       var title = {
          text: 'Cluster layout and distribution ratio'   
       };      
       var tooltip = {
          pointFormat: '{series.name}: <b>{point.percentage:.1f}%</b>'
       };
       var plotOptions = {
          pie: {
             allowPointSelect: true,
             cursor: 'pointer',
             dataLabels: {
                 enabled: true,
                format: '{point.percentage:.1f} %'      
             },
             showInLegend: true
          }
       };
       var series= [{
          type: 'pie',
          name: 'total share',
          data: [{
            name: 'East Asian restaurants',
            y: 0.19,
            color: '#FFFF00'
         },{
            name: 'South Asian restaurants',
            y: 0.14,
            color:'#FFB6C1'
         }, {
            name: 'Europe restaurants',
            y: 0.6,
            color: '#7FFFD4'
         }, {
            name: 'Others',
            y: 0.07,
            color: '#7B68EE'
         }]

       }];     

       var json = {};   
       json.chart = chart; 
       json.title = title;     
       json.tooltip = tooltip;  
       json.series = series;
       json.plotOptions = plotOptions;
       $('#show').highcharts(json);  
    });
    //ranking chart
    $(document).ready(function() {  
   var chart = {
      type: 'bar'
   };
   var title = {
      text: 'Ranking of numbers of restaurants'   
   };
   var subtitle = {
      text: 'At the national level'  
   };
   var xAxis = {
      categories: ['Mexican','Indian', 'Other'],
      title: {
         text: null
      }
   };
   var yAxis = {
      min: 0,
      title: {
         text: 'percentage)',
         align: 'high'
      },
      labels: {
         overflow: 'justify'
      }
   };
   var tooltip = {
      valueSuffix: ' millions'
   };
   var plotOptions = {
      bar: {
         dataLabels: {
            enabled: true
         }
      }
   };
  
   var credits = {
      enabled: false
   };
   
   var series= [{
         name: 'Ranking by number',
            data:  [19, 12,10],
	   		color:'#aaa'
        }
      
   ];     
      
   var json = {};   
   json.chart = chart; 
   json.title = title;   
   json.subtitle = subtitle; 
   json.tooltip = tooltip;
   json.xAxis = xAxis;
   json.yAxis = yAxis;  
   json.series = series;
   json.plotOptions = plotOptions;
   json.credits = credits;
   $('#container').highcharts(json);
  
});

    //rank chart
    $(document).ready(function() {  
       var chart = {
          type: 'bar'
       };
       var title = {
          text: 'Ranking of restaurants types'   
       };
       var subtitle = {
          text: 'At the national level'  
       };
       var xAxis = {
          categories: ['Chinese','Indian', 'Other','Mexican'],
          title: {
             text: null
          }
       };
       var yAxis = {
          min: 0,
          title: {
             text: 'percentage or unit',
             align: 'high'
          },
          labels: {
             overflow: 'justify'
          }
       };

       var plotOptions = {
          bar: {
             dataLabels: {
                enabled: true
             }
          }
       };

       var credits = {
          enabled: false
       };

       var series= [{
                name: 'Ranking by percentage',
                data: [38, 79, 78, 91],
                color:'#FFE4E1'
            },{
                name: 'Ranking by actual counts',
                data: [12, 19, 77, 10],
                color:'	#7B68EE'
            }

       ];     

       var json = {};   
       json.chart = chart; 
       json.title = title;   
       json.subtitle = subtitle; 
       json.xAxis = xAxis;
       json.yAxis = yAxis;  
       json.series = series;
       json.plotOptions = plotOptions;
       json.credits = credits;
       $('#rank').highcharts(json);

    });

             
                    
           
</script>

</body>
</html>
