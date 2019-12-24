<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="initial-scale=1, maximum-scale=1, user-scalable=no">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.6.0/dist/leaflet.css" integrity="sha512-xwE/Az9zrjBIphAcBb3F6JVqxf46+CDLwfLMHloNu6KEQCAWi6HcDUbeOfBIptF7tcCzusKFjFw2yuvEpDL9wQ==" crossorigin=""/>
    <script src="https://unpkg.com/leaflet@1.6.0/dist/leaflet.js" integrity="sha512-gZwIG9x3wUXg2hdXF6+rVkLF/0Vi9U8D2Ntg4Ga5I5BZpVkVxlJWbSQtXPSiUTtC0TjtGOmxa1AJPuV0CPthew==" crossorigin=""></script>
    <script src = "parks-polygon-representation.js"></script>
    <script src = "bikeways.js"></script>
    <link rel="stylesheet" href=leaflet_parks.css>
  </head>
  <body>
    <div id="map"></div>

    <script>

      var mbAttr = 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, ' +
          '<a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, ' +
          'Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
          mbUrl = 'https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NXVycTA2emYycXBndHRqcmZ3N3gifQ.rJcFIG214AriISLbB6B5aw'
        
      var grayscale = L.tileLayer(
        mbUrl, {
            id: 'mapbox/light-v9',
            attribution: mbAttr
          }),
        streets  = L.tileLayer(
          mbUrl, {
            id: 'mapbox/streets-v11',   
            attribution: mbAttr
          });



/*       L.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NXVycTA2emYycXBndHRqcmZ3N3gifQ.rJcFIG214AriISLbB6B5aw', {
        maxZoom: 18,
        attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, ' +
          '<a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, ' +
          'Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
        id: 'mapbox/light-v9'
      }).addTo(map); */

      // control that shows state info on hover

      function getBikeLineWeight(l) {
        let bwlw;
        l == "Local Street" ? bwlw = 1 :
        l == "Painted Lanes" ? bwlw = 2 : 
          bwlw = 3; 
        //console.log(bwlw);
        return bwlw;
      }

      function getColor(d) {
        return d > 100 ? '#800026' :
            d > 50 ? '#BD0026' :
            d > 35 ? '#E31A1C' :
            d > 20 ? '#FC4E2A' :
            d > 10   ? '#FD8D3C' :
            d > 5   ? '#FEB24C' :
            d > 2   ? '#FED976' :
                  '#FFEDA0';
      }

      // styles for layers
      function style(feature) {
        return {
          weight: 1,
          opacity: 1,
          color: 'white',
          //dashArray: '3',
          fillOpacity: 0.7,
          fillColor: getColor(feature.properties.area_ha)
        };
      }

      function style_bikeways(feature) {
        return {
          weight: getBikeLineWeight(feature.properties.bikeway_type),
          opacity: 1,
          color: 'green'
        };
      }

      function highlightFeature(e) {
        var layer = e.target;

        layer.setStyle({
          weight: 5,
          color: 'coral',
          dashArray: '',
          fillOpacity: 0.7
        });

        if (!L.Browser.ie && !L.Browser.opera && !L.Browser.edge) {
          layer.bringToFront();
        }

        info.update(layer.feature.properties);
      }

      var parks;

      function resetHighlight(e) {
        parks.resetStyle(e.target);
        info.update();
      }

      function resetHighlight_bikeway(e) {
        bikeways.resetStyle(e.target);
        info.update();
      }

      function zoomToFeature(e) {
        map.fitBounds(e.target.getBounds());
      }

      function onEachFeature(feature, layer) {
        layer.on({
          mouseover: highlightFeature,
          mouseout: resetHighlight,
          click: zoomToFeature
        });
      }

      function onEachFeature_bikeways(feature, layer) {
        layer.on({
          mouseover: highlightFeature,
          mouseout: resetHighlight_bikeway,
          click: zoomToFeature
        });
      }

      parks = L.geoJson(parks, {
        style: style,
        onEachFeature: onEachFeature
      });

      bikeways = L.geoJson(bikeways_js, {
        style: style_bikeways,
        onEachFeature: onEachFeature_bikeways
      });

      var map = L.map('map', {
        center: [49.247232, -123.127741], 
        zoom: 12,
        layers: [grayscale, parks, bikeways]
      });

      var legend = L.control({position: 'bottomright'});

      legend.onAdd = function (map) {

        var div = L.DomUtil.create('div', 'info legend'),
          grades = [0, 2, 5, 10, 20, 35, 50, 100],
          labels = [],
          from, to;

        for (var i = 0; i < grades.length; i++) {
          from = grades[i];
          to = grades[i + 1];

          labels.push(
            '<i style="background:' + getColor(from + 1) + '"></i> ' +
            from + (to ? '&ndash;' + to : '+'));
        }

        div.innerHTML = labels.join('<br>');
        return div;
      };

      legend.addTo(map);

      var baseLayers = {
        "Grayscale": grayscale,
        "Streets": streets
      }

      var overlays = {
        "Parks": parks,
        "Bikeways": bikeways
      } 
      var layers = L.control.layers(baseLayers, overlays, {position: 'bottomleft'})
      layers.addTo(map);

      var info = L.control();
      
      info.onAdd = function (map) {
        this._div = L.DomUtil.create('div', 'info');
        this.update();
        return this._div;
      };

      /* info.update = function (props) {
        this._div.innerHTML = '<h4>Feature Info </h4>' +  (props ?
          '<b>' + props.park_name + '</b><br />' + 'area: '+ props.area_ha
          : 'Hover over a park');
      }; */

      info.update = function (props) {
        if (props) {
          if (props.park_name) {
          this._div.innerHTML = '<h4>Feature Info </h4>' + 
          '<b>' + props.park_name + '</b><br />' + 'area: '+ props.area_ha
          } else if (props.bikeway_type) {
            this._div.innerHTML = '<h4>Feature Info </h4>' + 
            '<b>' + props.bike_route_name + '</b><br />' + 'Bikeway Type: '+ props.bikeway_type
          }
        } else {
          this._div.innerHTML = '<h4>Feature Info </h4>' + 'Hover over a feature';
        }
      };

      info.addTo(map);
  
    </script>
  </body>
</html>