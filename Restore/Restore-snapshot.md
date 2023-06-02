## 1) First we create a repository for our snapshots in the main cluster that has the indices:

    PUT /_snapshot/<repo name>
    {
      "type": "fs",
      "settings": {
        "location": "my_backup_location"
      }
    }

 exsample

    PUT /_snapshot/place_3
    {
      "type": "fs",
      "settings": {
        "location": "place_3"
      }
    }


we can use kibanas settings to create repos too, go to stack management  > Snapshot and Restore  and make one from there instead.


If the location of repo.path is set in the elasticsearch config file as for example: /usr/share/elasticsearch/backup  then when we create a repo, if we just specify a name, if would count as /usr/share/elasticsearch/backup/<repo name>  so we dont need to write the whole path.

## 2) Then we take a snapshot from the cluster that we want its data to be restored on another cluster:

    PUT /_snapshot/<repo name>/<snapshot name>?wait_for_completion=true
    {
      "indices": "data_stream_1,index_1,index_2",    # if we want all the indexes, dont write this line.
      "ignore_unavailable": true,
      "include_global_state": false
    }

exsample

    PUT /_snapshot/place_3/place_3?wait_for_completion=true
    {
      "indices": "place_3",    # if we want all the indexes, dont write this line.
      "ignore_unavailable": true,
      "include_global_state": false
    }

## 3) Then we copy the repo directory : /usr/share/elasticsearch/backup/<repo name>  to the other clusters nodes, where its repo.path is located to, and lets say its also the same as the other cluster.


# Check Index
    GET /_cat/indices

# To Restore Index , it is necessary that the index does not exist, and if it exists, it must be deleted.

    DELETE place_3

# Check Snapshot
    GET /_cat/snapshots




## 4) After copying the directory to the other clusters repo path (where it is shared between the nodes of that cluster) , we go into the other new cluster, and make a repository with the same name as we did with the first cluster, and then we can see the snapshot name for the created repo, we restore it like this :

    POST /_snapshot/<repo name>/<snapshot name>/_restore
    {
      "indices": "data_stream_1,index_1,index_2",   # if some indexes gives us an error, we then need to specify all the    other indexes here.
      "ignore_unavailable": true,
      "include_global_state": false,
      "include_aliases": false
    }

    POST /_snapshot/place_3/place_3/_restore
    {
      "indices": "data_stream_1,index_1,index_2",   # if some indexes gives us an error, we then need to specify all the    other indexes here.
      "ignore_unavailable": true,
      "include_global_state": false,
      "include_aliases": false
    }
