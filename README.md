# postgrescluster-with-2-users

```

db:
  aws: 
    key: AKIA2T3
    keysecret:  8h8YPAEOw
  spec:
    users:
    - name: darknetuser
      password:
        type: AlphaNumeric
      databases:
      - darknet
    - name: db2
      password:
        type: AlphaNumeric
      databases:
      - db2
      options: "SUPERUSER"
    # pgo tag is ubi8-pg_version-postgis_version-build_version
    # ref: https://access.crunchydata.com/documentation/postgres-operator/v5/references/components/#container-tags
    #image: registry.developers.crunchydata.com/crunchydata/crunchy-postgres-gis:ubi8-15.4-3.3-0
   # image: registry.developers.crunchydata.com/crunchydata/crunchy-postgres:ubi8-15.4-1
    # image: registry.developers.crunchydata.com/crunchydata/crunchy-postgres:ubi8-15.5-0
    postgresVersion: 15
    databaseInitSQL:
      key: init.sql
      name: '{{ include "postgres.fullname" $ }}-init-sql'
    patroni:
      dynamicConfiguration:
        postgresql:
          parameters:
          shared_preload_libraries: timescaledb
          pg_hba:
          - "hostnossl all all all md5"
    instances:
    - replicas: 1
      dataVolumeClaimSpec:
        accessModes:
        - "ReadWriteOnce"
        resources:
          requests:
            storage: "10Gi"
      # resources:
      #   limits:
      #     memory: 4Gi
    backups:
      enabled: true
      pgbackrest:
        image: registry.developers.crunchydata.com/crunchydata/crunchy-pgbackrest:ubi8-2.47-2
        configuration:
        - secret:
            name: '{{ include "postgres.fullname" $}}-s3'
            create: "true"
        global:
          repo1-path: /pgbackrest/postgres-operator/postgres/repo1 # change it the way you want!
          repo1-retention-full: "14"
          repo1-retention-full-type: time
        repos:
        - name: repo1
          schedules:
            full: "30 1 1,15 * *"
            incremental: "0 0 * * 0"
          s3:
            bucket: "terraform-yannick"
            endpoint: "s3.us-east-2.amazonaws.com"
            region: "us-east-2"
    monitoring: 
        pgmonitor:
          exporter: {} # this is for the latest version of the exporter
            # image: registry.developers.crunchydata.com/crunchydata/crunchy-postgres-exporter:ubi8-5.6.0-0
            # you can use above for a specific image version
```
