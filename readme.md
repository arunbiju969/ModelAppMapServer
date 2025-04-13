
---

# Middleton Map Application

This repository contains a MapServer-based web mapping application for visualizing geospatial data using OpenLayers. The application is designed to serve WMS and WFS layers and display them in a user-friendly web interface.

## Features

- **MapServer Configuration**: A my_service.map file is provided to configure WMS and WFS services for geospatial data layers.
- **OpenLayers Integration**: A web interface (`index.html`) is included to display the map and interact with the layers.
- **Layer Management**: Users can toggle layers on and off and view legends dynamically.
- **Support for Multiple Projections**: The application supports EPSG:4326, EPSG:3857, and EPSG:26920 projections.

---

## MapServer Configuration (`my_service.map`)

The my_service.map file defines the MapServer configuration for serving geospatial data. Key highlights include:

- **Service Metadata**:
  - WMS and WFS services are enabled with titles, supported projections, and request capabilities.
  - Example WMS URL: `http://127.0.0.1/cgi-bin/mapserv.exe?map=C:/ms4w/apps/ModelApp/my_service.map`

- **Layers**:
  - **Forest**: Displays forest polygons styled by age (e.g., young, mature, old growth).
  - **Roads**: Displays road lines with a red color.
  - **Buildings**: Displays building points with a square symbol.

- **Projections**:
  - The map and layers are configured to support multiple projections, including WGS84 (EPSG:4326) and NAD83 / UTM Zone 20N (EPSG:26920).

- **Styling**:
  - Layers are styled based on attributes (e.g., forest age) with custom colors and labels.

---

## Key Components of a Map File

A MapServer map file (e.g., `my_service.map`) is a configuration file that defines how geospatial data is served and styled. Below are the key components that a map file should have:

1. **MAP Object**:
   - The root object that contains all other elements.
   - Defines global settings such as the map's extent, projection, and image output format.

   Example:
   ```
   MAP
       NAME "MiddletonMap"
       EXTENT -180 -90 180 90
       PROJECTION
           "init=epsg:4326"
       END
       IMAGETYPE png
   END
   ```

2. **WEB Object**:
   - Configures web-specific settings, such as metadata for WMS/WFS services and error handling.

   Example:
   ```
   WEB
       METADATA
           "wms_title" "Middleton Map Service"
           "wms_srs" "EPSG:4326"
       END
   END
   ```

3. **LAYER Objects**:
   - Defines individual layers of geospatial data to be served.
   - Each layer specifies its data source, type (e.g., POINT, LINE, POLYGON), projection, and styling.

   Example:
   ```
   LAYER
       NAME "Forest"
       TYPE POLYGON
       DATA "C:/ms4w/apps/ModelApp/MtonData4QGIS/Forest.shp"
       PROJECTION
           "init=epsg:4326"
       END
       CLASS
           STYLE
               COLOR 34 139 34
           END
       END
   END
   ```

4. **PROJECTION Object**:
   - Specifies the coordinate reference system (CRS) for the map and its layers.
   - Typically uses EPSG codes (e.g., EPSG:4326 for WGS84).

5. **OUTPUTFORMAT Object**:
   - Defines the format of the map images (e.g., PNG, JPEG) or vector data (e.g., GeoJSON, GML).

   Example:
   ```
   OUTPUTFORMAT
       NAME "png"
       DRIVER "AGG/PNG"
       MIMETYPE "image/png"
   END
   ```

6. **SYMBOL Object** (Optional):
   - Defines custom symbols for styling layers, such as point markers or line patterns.

   Example:
   ```
   SYMBOL
       NAME "circle"
       TYPE ELLIPSE
       POINTS 1 1 END
       FILLED TRUE
   END
   ```

7. **INCLUDE Directive**:
   - Allows external files to be included in the map file for modularity.

   Example:
   ```
   INCLUDE "C:/ms4w/apps/ModelApp/includes/roads.map"
   ```

By including these components, a map file can effectively define and serve geospatial data through MapServer.

---

## Web Interface (`index.html`)

The index.html file provides a web-based interface for interacting with the map. Key features include:

- **OpenLayers Integration**:
  - The map is initialized with layers served by MapServer.
  - Layers include `Forest`, `Roads`, and `Buildings`.

- **Layer Switcher**:
  - A sidebar allows users to toggle layers on and off.

- **Dynamic Legend**:
  - Legends for each layer are dynamically fetched using the WMS `GetLegendGraphic` request.

- **Responsive Design**:
  - The interface is styled for full-screen display with a sidebar for controls and a main map area.

---

## Extent Calculation

The extent of the map was calculated based on the bounding box of the geospatial data layers included in the application. The following steps were used to determine the extent:

1. **Data Inspection**:
   - The spatial extent of each layer (e.g., Forest, Roads, Buildings) was inspected using GIS tools to identify their minimum and maximum coordinates.

2. **Bounding Box Aggregation**:
   - The individual extents of all layers were aggregated to determine the overall bounding box for the map.
   - The bounding box is defined by the minimum and maximum X (longitude) and Y (latitude) coordinates.

3. **Projection Consideration**:
   - The extent was calculated in the native projection of the data (e.g., EPSG:4326 or EPSG:26920) to ensure accuracy.

4. **MapServer Configuration**:
   - The calculated extent was set in the `EXTENT` parameter of the `my_service.map` file to define the visible area of the map.

### Code Used for Extent Calculation

The following command was used to reproject the data and inspect its extent:

```bash
ogr2ogr -f GeoJSON -t_srs EPSG:4326 /vsistdout/ C:\ms4w\apps\ModelApp\MtonData4QGIS\combined.vrt | ogrinfo -ro -so -al /vsistdin/
```

- **`ogr2ogr`**:
  - `-f GeoJSON`: Specifies the output format as GeoJSON.
  - `-t_srs EPSG:4326`: Reprojects the data to the WGS84 coordinate system (EPSG:4326).
  - `/vsistdout/`: Outputs the result to standard output (stdout).
  - `C:\ms4w\apps\ModelApp\MtonData4QGIS\combined.vrt`: The input file, which is a virtual raster table (VRT).

- **Pipe (`|`)**:
  - Passes the output of `ogr2ogr` directly to `ogrinfo`.

- **`ogrinfo`**:
  - `-ro`: Opens the data in read-only mode.
  - `-so`: Displays only the summary information of the layers.
  - `-al`: Lists all layers in the dataset.
  - `/vsistdin/`: Reads the input from standard input (stdin).

This command chain reprojects the data to EPSG:4326 and then inspects the reprojected data for its summary information.

---

## Creation of the Combined VRT File

The `combined.vrt` file was created to unify multiple geospatial layers into a single virtual dataset. This allows for easier management and analysis of the data. The following steps were taken to create the VRT file:

1. **Selection of Layers**:
   - The layers included in the VRT file are:
     - `forest.shp`: Represents forest polygons.
     - `MtonRoads.shp`: Represents road lines.
     - `MtonBldgs.shp`: Represents building points.

2. **VRT File Structure**:
   - The VRT file uses the `<OGRVRTUnionLayer>` element to combine the selected layers into a single virtual layer named `combined`.
   - Each layer is defined using the `<OGRVRTLayer>` element, specifying the source data file and layer name.

3. **File Path Configuration**:
   - The paths to the source shapefiles are relative to the location of the VRT file, ensuring portability.

4. **Purpose**:
   - The `combined.vrt` file simplifies the process of querying and visualizing multiple layers as a single dataset.

### Example VRT File

Below is the content of the `combined.vrt` file:

```xml
<OGRVRTDataSource>
    <OGRVRTUnionLayer name="combined">
        <OGRVRTLayer name="forest">
            <SrcDataSource>forest.shp</SrcDataSource>
            <SrcLayer>forest</SrcLayer>
        </OGRVRTLayer>
        <OGRVRTLayer name="roads">
            <SrcDataSource>MtonRoads.shp</SrcDataSource>
            <SrcLayer>MtonRoads</SrcLayer>
        </OGRVRTLayer>
        <OGRVRTLayer name="buildings">
            <SrcDataSource>MtonBldgs.shp</SrcDataSource>
            <SrcLayer>MtonBldgs</SrcLayer>
        </OGRVRTLayer>
    </OGRVRTUnionLayer>
</OGRVRTDataSource>
```

This file can be edited to include additional layers or modify existing ones as needed.

---

## How to Use

1. **Set Up MapServer**:
   - Install MapServer and configure it to serve the my_service.map file.
   - Ensure the paths in the my_service.map file are correct for your environment.

2. **Run the Web Application**:
   - Place the index.html file in a web-accessible directory.
   - Open the file in a browser to view the map.

3. **Interact with the Map**:
   - Use the sidebar to toggle layers and view legends.
   - Pan and zoom the map to explore the data.

---

## Example URLs

- **WMS GetCapabilities**:
  ```
  http://127.0.0.1/cgi-bin/mapserv.exe?map=C:/ms4w/apps/ModelApp/my_service.map&SERVICE=WMS&REQUEST=GetCapabilities
  ```

- **WFS GetCapabilities**:
  ```
  http://127.0.0.1/cgi-bin/mapserv.exe?map=C:/ms4w/apps/ModelApp/my_service.map&SERVICE=WFS&REQUEST=GetCapabilities
  ```

---

## Requirements

- **MapServer**: Ensure MapServer is installed and configured.
- **Web Server**: A web server (e.g., Apache) to serve the index.html file.
- **Browser**: A modern web browser to view the application.

---