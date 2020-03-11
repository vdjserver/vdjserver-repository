Import data into VDJServer Repository
=====================================

Currently the process is manual as study metadata structure and
analysis provenance are still undergoing development. The python
scripts to perform the import are located in the vdjserver-repair
repository, and they need to be run on the repository node as they
connect directly with the mongodb docker container.

The scripts do not directly import the data into the database, instead
they create a file containing a Javascript program which is run inside
mongodb to create the data.

These instructions are for the AIRR Data Commons repository. There are
also scripts for iReceptor but that is expected to be deprecated once
the AIRR API is published.

There are basic two pieces to be imported:

* Study metadata in a denormalized structure which includes MiAIRR
  data elements but also VDJServer specific metadata. This gets put
  into the "repertoire" collection.

* Set of AIRR TSV files containing rearrangement annotations
  associated with the study. This gets put into the "rearrangement"
  collection.

IgBlast is the main tool to produce the AIRR TSV files in
VDJServer. While we might also want to load analysis data from
RepCalc, that is for the future. Here is the information that needs to
be gathered:

* The UUID for the VDJServer project.

* The UUID for the IgBlast job(s). A project might have multiple
  IgBlast jobs if it contains both T-cell and B-cell data, for
  example.

The two scripts in vdjserver-repair repository are:

* airr_project_import.py - this imports the study metadata.

* airr_file_import.py - this imports a single AIRR TSV file.

Here are the basic steps:

* Login to vdj-rep-01.tacc.utexas.edu with vdj account, `source
  ~/vdjserver.env` to setup the environment, and `cd
  /var/www/docker/vdjserver/vdjserver-repair`. Use
  `auth-tokens-refresh` or `auth-tokens-create` to get Agave token for
  doing command line operations.

* Download the agave job archive directory with all of the job output
  files. If the project has been moved to Community Data, which
  presumably it has already in order to be public, the normal
  `jobs-output-get` command will not work because the files have been
  moved from the original archive path. Instead use `files-get` and
  replace the top-level path `/projects` with `/community`. This
  directory is the `archivePath` attribute in the job metadata which
  can be viewed with `jobs-list -v uuid`. (Special) vdj-rep-01 has
  VDJServer's storage directly mounted so you can copy the files
  directory from `/vdjZ` instead of going through Agave.

* You should now have a local directory with all the job files. Verify
  that `process_metadata.json` exists and that you see all of the AIRR
  TSV files you expect. The AIRR TSV files are compressed and need to
  be expanded with a command like `ls *.airr.tsv.zip | xargs -n 1
  unzip`

* Now run `python airr_project_import.py -p project_uuid -j job_uuid`
  to generate the import Javascript program. It puts the file at
  `/vdjZ/mongodb/tmp/study.js`

* Run the Javascript within mongodb. As study metadata is small it
  should run quickly. It's a good idea to delete the file afterward as
  it contains authentication information.

```
docker exec -it vdjr-mongo mongo --authenticationDatabase admin v1airr /data/db/tmp/study.js
rm -f /vdjZ/mongodb/tmp/study.js
```

* Now import each of the AIRR TSV files individually. Run `python
  airr_file_import.py -p project_uuid -t tsv_path` to generate the
  import Javascript program. Make sure to pass the `zip` file name for
  the AIRR TSV because that is the exact name in
  `process_metadata.json`; it will remove `zip` when reading the
  data. These files can be quite large, so multiple Javascript files
  (file0.js, file1.js, etc.)  will be created to not exceed the
  mongodb limit. They are placed in `/vdjZ/mongodb/tmp`. Make sure to
  run these files in the correct order, and delete them all so they
  are not accidently re-run for another file.

```
docker exec -it vdjr-mongo mongo --authenticationDatabase admin v1airr /data/db/tmp/file0.js
docker exec -it vdjr-mongo mongo --authenticationDatabase admin v1airr /data/db/tmp/file1.js
docker exec -it vdjr-mongo mongo --authenticationDatabase admin v1airr /data/db/tmp/file2.js
rm -f /vdjZ/mongodb/tmp/file*.js
```

* If you have a large project with many AIRR TSV files, you might want
  to write a shell script to run all the commands in a batch. It can
  take from hours to days to load all the data for a project.

* Lastly, run a query command against the REST API to verify the data
  is accessible.
