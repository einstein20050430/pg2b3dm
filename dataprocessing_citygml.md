# Dataprocessing CityGML -> PolyhedralSurfaceZ

pg2b3dm takes as input triangulated PolyhedralSurfaceZ geometries in PostGIS. In this document a sample 
workflow is described for processing CityGML files using FME, GDAL and PostGIS.

Sample input files: https://www.opengeodata.nrw.de/produkte/geobasis/3dg/lod2_gml/

### FME
![FME_CityGML_Shp](https://user-images.githubusercontent.com/9533288/77912263-83c8fb80-7292-11ea-8b20-31b775ccc545.PNG)
In FME the *CityGML* file is imported and the buildings are exported as triangulated *shapefiles* using the following steps:

1. *Import CityGML* open the file using the CityGML reader and select the parts from the CityGML you want to use.
2.  *Select the Solids* using the GeometryFilter. 
3.  *Triangulate* the solids using the Triangulator, pg2b3dm needs PolyhedralSurfaceZ with 4 coordinates. 
4.  *Export as SHP* using the Esri shapefile writer.

### GDAL
5. Import SHP as PolyhedralSurface using ogr2ogr. 
	`-dim 3` makes sure to use the x, y and z information
	 `-nlt POLYHEDRALSURFACEZ` sets the correct geometry
```
$ ogr2ogr -f "PostgreSQL" "PG:host=server user=username dbname=database" $f -nln schema.tablename -dim 3 -nlt POLYHEDRALSURFACEZ
```

### PostGIS
6.  Delete geometry collections containing geometries with more or less than 4 vertices (triangles).
```
psql> DELETE FROM schema.tablename WHERE (ST_Npoints(wkb_geometry)::float/ST_NumGeometries(wkb_geometry)::float) != 4;
```
7. Add id and color columns, both in text format.
```
psql> ALTER TABLE schema.tablename ADD COLUMN id text;
psql> ALTER TABLE schema.tablename ADD COLUMN color text;
```
8. Populate the added columns.
```
psql> UPDATE schema.tablename SET id = ogc_fid;
psql> UPDATE schema.tablename SET color = '#f5f5f5';
```

Now it's possible to run pg2b3dm!

Result looks like: 
![Duisburg_pg2b3dm](https://user-images.githubusercontent.com/9533288/77912264-862b5580-7292-11ea-8758-1aa1895c249f.PNG)

Live demo: https://geodan.github.io/pg2b3dm/sample_data/duisburg/mapbox/#15.62/51.430166/6.782675/0/45

