# edit docker image metadata

The initial motiviation for the creation of the tool came from
the fact that it is not possible to remove VOLUME entries in
an image. You can basically change a USER or WORKDIR setting
but you can only ever add VOLUME and PORT entries.

The wish to REMOVE ALL VOLUMES came from the fact that I did
want to download a tested image for local tests where the
data part should be committed to the history as well in order
to turn back both program and data to a defined state so that 
another test run will start off the exact same checkpoint.

While docker does not allow to edit the metadata of an image
directly, there is a workaround - one may "docker save" an
image into an archive file that contains all the layers and
metadata json files. After modifying the content one can
"docker load" the result back with the history being preserved.

Correcting some images from other sources became such a
regular task that I started to fill in a python script to
help with the daily work. In order to allow coworkers to
understand what was intended, the input syntax is somewhat
descriptive (likeSQL).

     ./docker-copyedit.py \
     FROM image1 INTO image2 REMOVE ALL VOLUMES
     
     ./docker-copyedit.py \
     into image2 from image1 set user null
     
     ./docker-copyedit.py \
     set user null and set cmd null from image1 into image2
     
     ./docker-copyedit.py FROM image1 INTO image2 \
         set user null + set cmd null + rm all volumes
         
     ./docker-copyedit.py FROM image1 INTO image2 -vv \
         REMOVE VOLUME /var/atlassian/jira-data

     ./docker-copyedit.py FROM image1 INTO image2 -vv \
         set entrypoint null and set cmd /entrypoint.sh

Of course you may have image1 and image2 to be the same
tag name but remember that the image hash value will 
change wile copyediting the image archive on the disk.
You will be left with a dangling old (untagged) image.

By default the tool will use a local "load.tmp" temporary
directory. You may set "-t $TMPDIR" explicitly to have it
run in a normal temporary directory - but be aware that
the archive files of save/load can be quite big and the
tool will even unpack the archive temporarily. That's why
the "-t tmpdir" should point to a space that is hopefully
big enough (like the workspace you are already in).
