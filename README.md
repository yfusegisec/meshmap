# meshmap
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="initial-scale=1,maximum-scale=1,user-scalable=no" />
    <title>meshmap</title>

    <link rel="stylesheet" href="https://js.arcgis.com/4.30/esri/themes/light/main.css" />
    <script src="https://js.arcgis.com/4.30/"></script>
    <style>
      html,
      body,
      #map{
	height:80vh;
	width:80vw;
      }
      #viewDiv {
        padding: 0;
        margin: 0;
        height: 100%;
        width: 100%;
      }
    </style>
    <script>

 
    </script>
    <script>
      require(["esri/Map", 
	       "esri/WebMap",
               "esri/views/MapView", 
	       "esri/widgets/Legend",
               "esri/layers/FeatureLayer",
	       "esri/Graphic",
               "esri/geometry/SpatialReference",
               "esri/geometry/Point",
	       "esri/geometry/Extent",
	       "esri/geometry/coordinateFormatter",
	       "esri/geometry/support/geodesicUtils",
               "esri/widgets/Popup"
              ], (
		Map, 
		WebMap,
		MapView, 
		Legend,
		FeatureLayer, 
		Graphic, 
		SpatialReference, 
		Point, 
		Extent,
		coordinateFormatter,
		geodesicUtils,
		Popup
		) => {
      	const MIL = 1000000;
	const ZOOMLEVEL12=72223.819286;
        let centerx = 139.7286691+Math.random();
        let centery = 35.692726+Math.random();
        let xminCoor = centerx-(Math.random()*MIL*0.1)/MIL;
        let xmaxCoor = centerx+(Math.random()*MIL*0.1)/MIL;
        let yminCoor = centery-(Math.random()*MIL*0.1)/MIL;
        let ymaxCoor = centery+(Math.random()*MIL*0.1)/MIL;

	//*****手打ちで座標を指定する場合は以下*****
	//yminCoor = 45.959774187575626;
	//ymaxCoor = 46.14372211019538;
	//xminCoor = 137.84262861015588;
        //xmaxCoor = 137.94533282864577;
	//*******************************************
	
        console.log("経度最小値："+xminCoor)
        console.log("経度最大値："+xmaxCoor)
        console.log("緯度最小値："+yminCoor)
        console.log("緯度最大値："+ymaxCoor)
        console.log("経度幅："+(xmaxCoor-xminCoor))
        console.log("緯度幅："+(ymaxCoor-yminCoor))
        const map = new WebMap({
          //basemap: "hybrid"//3D
          basemap: "topo-vector"//2D
        });

        const view = new MapView({
          container: "viewDiv",
          map: map,
	  container: "viewDiv", 
	  //zoom: 1
          //extent: {
            // autocasts as new Extent()
            //xmin: 15551190,
            //ymin: 4251190,
            //xmax: 15559190,
            //ymax: 4259190,
	      //spatialReference: 3857
        //spatialReference: 102100
	      //spatialReference: SpatialReference.WGS84
          //}
        });

      const centerPoint = new Point({
	x:xminCoor+(xmaxCoor-xminCoor)/2,
	y:yminCoor+(ymaxCoor-yminCoor)/2,
        //spatialReference: SpatialReference.WGS84//wkid4326
      });
      //view.center = centerPoint;
        const point_center = {
          type: "point", // autocasts as new Point()
          longitude: centerPoint.x,
          latitude: centerPoint.y
        };

      let defaultExtent = new Extent({
        xmin: xminCoor,
        xmax:xmaxCoor,
        ymin:yminCoor,
        ymax:ymaxCoor,
        spatialReference: SpatialReference.WGS84
     });
	let extentSizeRate=(defaultExtent.height*MIL)/(defaultExtent.width*MIL);
	//緯度差、経度差を実際の距離に変換してからextentの高さ/幅の比率を計算
        //extentで定めた最小・最大の地点 
        const p_xymin = new Point({
          longitude: defaultExtent.xmin,
          latitude: defaultExtent.ymin,
        });
        const p_xminymax = new Point({
          longitude: defaultExtent.xmin,
          latitude: defaultExtent.ymax,
        });
        const p_xmaxymin = new Point({
          longitude: defaultExtent.xmax,
          latitude: defaultExtent.ymin,
        });
        const p_xymax = new Point({
          longitude: defaultExtent.xmax,
          latitude: defaultExtent.ymax,
        });
var distanceUnits = "kilometers";
var latiDistanceResult = geodesicUtils.geodesicDistance(p_xymin, p_xminymax, distanceUnits);
var longDistanceResult = geodesicUtils.geodesicDistance(p_xymin, p_xmaxymin, distanceUnits);
console.log("垂直方向距離:"+latiDistanceResult.distance+"km");
console.log("水平方向距離:"+longDistanceResult.distance+"km");
    //緯度の差分の幅と経度の差分の幅を比較して前者の幅の比率が後者と比較し必要に応じてxmaxを移動
    let coorLate = (latiDistanceResult.distance*MIL)/(longDistanceResult.distance*MIL)
    console.log(coorLate)
    if(coorLate<=1.5){
   	defaultExtent.xmax=defaultExtent.xmin+defaultExtent.height*1.5;
    }else{
   	defaultExtent.xmax=defaultExtent.xmin+defaultExtent.height*coorLate;
    }
    console.log(defaultExtent.xmax)
    view.extent = defaultExtent;

    //extentの範囲内でランダムに(x,y)を発生させる
    const randomPlot = 10;
    let randx=[];
    let randy=[];
    for(let i=0;i<randomPlot;i++){
      let rx=xminCoor;
      let ry=yminCoor;
      rx=Math.random()/10+xminCoor;
      ry=Math.random()/10+yminCoor;
      if(rx>xmaxCoor){
        rx=xminCoor+Math.floor((xmaxCoor-xminCoor)/(i+1));
      }
      if(ry>ymaxCoor){
        ry=yminCoor+Math.floor((ymaxCoor-yminCoor)/(i+1));
      }
      randx.push(rx);
      randy.push(ry);
    }

	//******add shape for map******

        /*************************
         * Create a point graphic
         *************************/
        const point_xymin = {
          type: "point", // autocasts as new Point()
          longitude: defaultExtent.xmin,
          latitude: defaultExtent.ymin,
        };
        const point_xminymax = {
          type: "point", // autocasts as new Point()
          longitude: defaultExtent.xmin,
          latitude: defaultExtent.ymax,
        };
        const point_xmaxymin = {
          type: "point", // autocasts as new Point()
          longitude: defaultExtent.xmax,
          latitude: defaultExtent.ymin,
        };
        const point_xymax = {
          type: "point", // autocasts as new Point()
          longitude: defaultExtent.xmax,
          latitude: defaultExtent.ymax,
        };
	
        //***********************************
        //ランダムにプロットする地点　×　10
        const point_random = {
          type: "point", // autocasts as new Point()
          longitude: randx[0],
          latitude: randy[0]
        };
        const point_random2 = {
          type: "point", // autocasts as new Point()
          longitude: randx[1],
          latitude: randy[1]
        };
        const point_random3 = {
          type: "point", // autocasts as new Point()
          longitude: randx[2],
          latitude: randy[2]
        };
        const point_random4 = {
          type: "point", // autocasts as new Point()
          longitude: randx[3],
          latitude: randy[3]
        };
        const point_random5 = {
          type: "point", // autocasts as new Point()
          longitude: randx[4],
          latitude: randy[4]
        };
        const point_random6 = {
          type: "point", // autocasts as new Point()
          longitude: randx[5],
          latitude: randy[5]
        };
        const point_random7 = {
          type: "point", // autocasts as new Point()
          longitude: randx[6],
          latitude: randy[6]
        };
        const point_random8 = {
          type: "point", // autocasts as new Point()
          longitude: randx[7],
          latitude: randy[7]
        };
        const point_random9 = {
          type: "point", // autocasts as new Point()
          longitude: randx[8],
          latitude: randy[8]
        };
        const point_random10 = {
          type: "point", // autocasts as new Point()
          longitude: randx[9],
          latitude: randy[9]
        };
        
        //***********************************
        const centerMarkerSymbol = {
          type: "simple-marker", 
          color: [200, 0, 40],
          outline: {
            // autocasts as new SimpleLineSymbol()
            color: [255, 255, 255],
            width: 2
          }
        };
        const markerSymbol = {
          type: "simple-marker", 
          color: [226, 119, 40],
          outline: {
            // autocasts as new SimpleLineSymbol()
            color: [255, 255, 255],
            width: 2
          }
        };
        const randomMarkerSymbol = {
          type: "simple-marker", 
          color: [0, 255, 0],
          outline: {
            // autocasts as new SimpleLineSymbol()
            color: [255, 255, 255],
            width: 2
          }
        };

        // Create a graphic and add the geometry and symbol to it
        const centerPointGraphic = new Graphic({
          geometry: point_center,
          symbol: centerMarkerSymbol
        });
        const xyminPointGraphic = new Graphic({
          geometry: point_xymin,
          symbol: markerSymbol
        });
        const xminymaxPointGraphic = new Graphic({
          geometry: point_xminymax,
          symbol: markerSymbol
        });
        const xmaxyminPointGraphic = new Graphic({
          geometry: point_xmaxymin,
          symbol: markerSymbol
        });
        const xymaxPointGraphic = new Graphic({
          geometry: point_xymax,
          symbol: markerSymbol
        });
        const randomPointGraphic = new Graphic({
          geometry: point_random,
          symbol: randomMarkerSymbol
        });
        const randomPoint2Graphic = new Graphic({
          geometry: point_random2,
          symbol: randomMarkerSymbol
        });
        const randomPoint3Graphic = new Graphic({
          geometry: point_random3,
          symbol: randomMarkerSymbol
        });        
        const randomPoint4Graphic = new Graphic({
          geometry: point_random4,
          symbol: randomMarkerSymbol
        });
        const randomPoint5Graphic = new Graphic({
          geometry: point_random5,
          symbol: randomMarkerSymbol
        });
        const randomPoint6Graphic = new Graphic({
          geometry: point_random6,
          symbol: randomMarkerSymbol
        });
        const randomPoint7Graphic = new Graphic({
          geometry: point_random7,
          symbol: randomMarkerSymbol
        });
        const randomPoint8Graphic = new Graphic({
          geometry: point_random8,
          symbol: randomMarkerSymbol
        });        
        const randomPoint9Graphic = new Graphic({
          geometry: point_random9,
          symbol: randomMarkerSymbol
        });
        const randomPoint10Graphic = new Graphic({
          geometry: point_random10,
          symbol: randomMarkerSymbol
        });
        /*************************
         * Create a polygon graphic
         *************************/

        // Create a polygon geometry
	const rings = [
		[
		 [defaultExtent.xmin, defaultExtent.ymin],
		 [defaultExtent.xmax, defaultExtent.ymin],
		 [defaultExtent.xmax, defaultExtent.ymax],
		 [defaultExtent.xmin, defaultExtent.ymax],
		]
	];
        const polygon = {
          type: "polygon", // autocasts as new Polygon()
          rings:rings
        };


        // Create a symbol for rendering the graphic
        const fillSymbol = {
          type: "simple-fill", // autocasts as new SimpleFillSymbol()
          color: [255, 139, 79, 0.5],
          outline: {
            // autocasts as new SimpleLineSymbol()
            color: [255, 255, 255],
            width: 1
          },
        };

        // Add the geometry and symbol to a new graphic
        const extentAreaPolygonGraphic = new Graphic({
          geometry: polygon,
          symbol: fillSymbol,
          popupTemplate: {
           content:extentPopupContent,
           title: "Extent popup",
           location: { latitude:defaultExtent.ymax, longitude:defaultExtent.xmin }
	  }
        });

	function extentPopupContent(){
          const div = document.createElement("div");
	  const score = Math.random()*10;
          //const upArrow =
            '<svg width="16" height="16" ><polygon points="14.14 7.07 7.07 0 0 7.07 4.07 7.07 4.07 16 10.07 16 10.07 7.07 14.14 7.07" style="fill:green"/></svg>';
          //const downArrow =
            '<svg width="16" height="16"><polygon points="0 8.93 7.07 16 14.14 8.93 10.07 8.93 10.07 0 4.07 0 4.07 8.93 0 8.93" style="fill:red"/></svg>';
          div.innerHTML =
	      "<b>経度：</b>"+"<b>"+defaultExtent.xmin+"～"+defaultExtent.xmax+"</b><br/>"+
	      "<b>緯度：</b>"+"<b>"+defaultExtent.ymin+"～"+defaultExtent.ymax+"</b><br/>"+
              "<span style='color: " +
              (score < 5 ? "red" : "green") +
              ";'>score:" +Math.floor(score)+
              "</span>";
		return div;
	}

        //***********************************
        // Add the graphics to the view's graphics layer
        view.graphics.addMany([
          centerPointGraphic, 
          xyminPointGraphic, 
          xmaxyminPointGraphic, 
          xminymaxPointGraphic, 
          xymaxPointGraphic, 
          randomPointGraphic,
          randomPoint2Graphic,
          randomPoint3Graphic,
          randomPoint4Graphic,
          randomPoint5Graphic,
          randomPoint6Graphic,
          randomPoint7Graphic,
          randomPoint8Graphic,
          randomPoint9Graphic,
          randomPoint10Graphic,
	  extentAreaPolygonGraphic
        ]);
      });

    </script>
  </head>

  <body>
    <div id="map">
	<div id="viewDiv"></div>
    </div>
  </body>
</html>
