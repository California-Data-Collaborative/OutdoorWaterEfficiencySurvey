# Outdoor Water Efficiency Survey

A household survey in Southern California to estimate current outdoor water use habits, environmental attitudes, and penetration rates of different landscaping types.

This project is a collaboration between [CivicSpark](http://civicspark.lgc.org/), the [CaDC](http://californiadatacollaborative.com/), and participating utilities including the [Santa Ana Watershed Project Authority](http://www.sawpa.org/) (SAWPA), the [Inland Empire Utilities Agency](https://www.ieua.org/) (IEUA) and the [Eastern Municipal Water District](http://www.emwd.org/) (EMWD).

#### CivicSpark Fellows
* [Paul Caporaso](http://civicspark.lgc.org/our-fellows/entry/2645/)
* [Anna Garcia](http://civicspark.lgc.org/our-fellows/entry/2664/)
* [Steven Kerns](http://civicspark.lgc.org/our-fellows/entry/2674/)
* [Ana Patricia Lopez](http://civicspark.lgc.org/our-fellows/entry/2679/)
* [Abbey Pizel](http://civicspark.lgc.org/our-fellows/entry/2689/)
* [Amanda Schallert](http://civicspark.lgc.org/our-fellows/entry/2693/)

#### CaDC Staff
* Patrick Atwater
* Christopher Tull


## Methodology 

### Survey Sample Generation

Code to generate the survey sample is located in the [notebooks/create_sample](notebooks/create_sample.md) file.

## Data Sources 

* [Santa Ana Watershed boundary polygon](http://www.sawpa.net/Downloads/gis_layers.zip)
* [California census block group polygons](https://www.census.gov/cgi-bin/geo/shapefiles/index.php?year=2016&layergroup=Block+Groups)
* [California parcel boundaries](http://egis3.lacounty.gov/dataportal/2015/09/11/california-statewide-parcel-boundaries/)

All shapefiles have been reprojected to [EPSG 3310](http://spatialreference.org/ref/epsg/nad83-california-albers/) using a process like the following in PostGIS:

```sql
--After loading shapefile, set the current column projection
SELECT UpdateGeometrySRID('census_blockgroup_polygons_2016', 'geom', 4269);

ALTER TABLE census_blockgroup_polygons_2016 ADD COLUMN geom_3310 geometry(MultiPolygon,3310);

UPDATE census_blockgroup_polygons_2016 SET geom_3310 = ST_Transform(geom, 3310)
WHERE ST_SRID(geom) = 4269;
```