# Nominatim Docker (Nominatim version 3.3)

1. Build
  ```
  docker build -t a-hahn/nominatim .
  ```
2. OSM data is located in /home/ubuntu/osm

3. Initialize Nominatim Database

  Build instructions for germany (runs ~ 1 day)
  Build for europe takes ~ 3 days and ~500 GB database size

  ```
  sudo docker run -t -v nominatim-germany-data:/data -v /home/ubuntu/osm:/osm  a-hahn/nominatim  sh /app/init.sh /osm/germany-latest.osm.pbf / 16
  ```
  In contrast to the original we use a volume to store the Postgres database (nominatim-germany-data) and we need to specify the
  map folder which is in /home/ubuntu/osm containing all the .pbf files for the various geomapping apps.
  '/' means 'from root' where 'data' ist located and 16 is the number of threads used.

4. After the import is finished the /home/me/nominatimdata/postgresdata folder will contain the full postgress binaries of
   a postgis/nominatim database. The easiest way to start the nominatim as a single node is the following:
   
```
sudo docker run \
-v nominatim-europe-data:/var/lib/postgresql/11/main \
-p 6432:5432 -p 7070:8080 \
--restart=always \
-d --name nominatim \
a-hahn/nominatim bash /app/start.sh

## or Europe on different file mount (about ~500 GB size) !
## sudo docker run --restart=always -p 6432:5432 -p 7070:8080 -d --name nominatim -v /media/sdb/docker/volumes/nominatim-europe-data:/var/lib/postgresql/11/main a-hahn/nominatim bash /app/start.sh
```

# Optional

**NOTE The (optional) next steps need to be adopted to the file structure above**


5. Advanced configuration. If necessary you can split the osm installation into a database and restservice layer

   In order to set the  nominatib-db only node:

   ```
   docker run --restart=always -p 6432:5432 -d -v /home/me/nominatimdata/postgresdata:/var/lib/postgresql/11/main nominatim sh /app/startpostgres.sh
   ```
   After doing this create the /home/me/nominatimdata/conf folder and copy there the docker/local.php file. Then uncomment the following line:

   ```
   @define('CONST_Database_DSN', 'pgsql://nominatim:password1234@192.168.1.128:6432/nominatim'); // <driver>://<username>:<password>@<host>:<port>/<database>
   ```

   You can start the  nominatib-rest only node with the following command:

   ```
   docker run --restart=always -p 7070:8080 -d -v /home/me/nominatimdata/conf:/data nominatim sh /app/startapache.sh
   ```

6. Configure incremental update. By default CONST_Replication_Url configured for Monaco.
If you want a different update source, you will need to declare `CONST_Replication_Url` in local.php. Documentation [here] (https://github.com/openstreetmap/Nominatim/blob/master/docs/Import-and-Update.md#updates). For example, to use the daily country extracts diffs for Gemany from geofabrik add the following:
  ```
  @define('CONST_Replication_Url', 'http://download.geofabrik.de/europe/germany-updates');
  ```

  Now you will have a fully functioning nominatim instance available at : [http://localhost:7070/](http://localhost:7070). Unlike the previous versions
  this one does not store data in the docker context and this results to a much slimmer docker image.


# Update

Full documentation for Nominatim update available [here](https://github.com/openstreetmap/Nominatim/blob/master/docs/admin/Import-and-Update.md#updates). For a list of other methods see the output of:
  ```
  docker exec -it nominatim sudo -u postgres ./src/build/utils/update.php --help
  ```

The following command will keep your database constantly up to date:
  ```
  docker exec -it nominatim sudo -u postgres ./src/build/utils/update.php --import-osmosis-all
  ```
If you have imported multiple country extracts and want to keep them
up-to-date, have a look at the script in
[issue #60](https://github.com/openstreetmap/Nominatim/issues/60).
