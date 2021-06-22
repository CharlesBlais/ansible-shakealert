Earthworm
=========

Setup an earthworm server

Requirements
------------

No requirements

Role Variables
--------------

* earthworm_uname = the earthworm user name
* earthworm_group = the earthworm group
* earthworm_gid = the gid of the group
* earthworm_download = the URL location of the earthworm download
* earthworm_src = location of the uncompressed source, make sure that matches the content of the tgz file

The following are earthworm application specific:

* earthworm_inst_id = the institute ID as defined in earthworm
* earthworm_directory = the earthworm directory
* earthworm_directory_bin (DO NOT MODIFY) = the bin location
* earthworm_directory_params (DO NOT MODIFY) = the params location
* earthworm_directory_log (DO NOT MODIFY) = the log location
* earthworm_log_retention = earthworm logs auto-clean after x days

Note: the items identified with DO NO MODIFY is because they match the default structure identified under ew_linux.bash.  If they are different, role must be modified.

The following are optional:

* earthworm_slink2ew = a dictionary with keys host, port, stream for each slink2ew process
* earthworm_ew2ringserver = a dictionary with keys host and port for each ew2ringserver process

Dependencies
------------

No dependencies

Example Playbook
----------------

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
