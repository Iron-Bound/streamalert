Schemas
=======

Overview
--------

Log schemas are required by StreamAlert to detect the correct log type for an incoming record.

Schemas are defined in ``conf/logs.json`` and used by rules to determine which records are analyzed.

They represent the structure of a given log in the form of key/value pairs.

Each key is the name of the field to be referenced in rules, and the value is data type the field is cast into.

Example log schema::

  {
    "example_log_name": {
      "parser": "parser-type-goes-here",
      "schema": {
        "field_1": "string",
        "field_2": "integer",
        "field_3": "boolean",
        "field_4": "float",
        "field_5": [],
        "field_6": {}
      }
    }
  }

Example rule:

.. code-block:: python

  @rule(logs=['example_log_name'],              # the log_name as defined above
        matchers=[],
        outputs=['slack'])
  def example_rule(rec):
    return (
      rec['field_1'] == 'string-value' and      # fields as defined in the schema above
      rec['field_2'] < 5 and
      'random-key-name' in rec['field_6']
    )


Options
-------

=============     ========     ======================
Key               Required     Description
-------------     ---------    ----------
``parser``        ``true``     The name of the parser to use for a given log's data-type.   Options include ``json, json-gzip, csv, kv, or syslog``
``schema``        ``true``     A map of key/value pairs of the name of each field with its type
``hints``         ``false``    A map of key/value pairs to assist in classification
``configuration`` ``false``    Options for parsing nested types, delimiters/separators, and more
=============     =========    ======================


Writing Schemas
---------------
Schema values are strongly typed and enforced.

Supported types:

* ``string`` - ``'example'``
* ``integer`` - ``0``
* ``boolean`` - ``true`` or ``false``
* ``float`` - ``0.0``

Special types:

* ``{}`` - zero or more key/value pairs of any type
* ``[]`` - zero or more elements of any type

Schemas can be as tight or as loose as you want (see Example: osquery).  Usage of the special types normally indicates a loose schema, in that not every part of the incoming data is described.


Hints
-----

explain hints here

JSON Examples
-------------

CloudWatch
~~~~~~~~~~

Example Log::

  {
    "version": "0",
    "id": "6a7e8feb-b491-4cf7-a9f1-bf3703467718",
    "detail-type": "EC2 Instance State-change Notification",
    "source": "aws.ec2",
    "account": "111122223333",
    "time": "2015-12-22T18:43:48Z",
    "region": "us-east-1",
    "resources": [
      "arn:aws:ec2:us-east-1:123456789012:instance/i-12345678"
    ],
    "detail": {
      "instance-id": "i-12345678",
      "state": "terminated"
    }
  }

Schema::

  "cloudwatch:ec2_event": {               # log name
    "schema": {
      "version": "string",                # type checking
      "id": "string",
      "detail-type": "string",
      "source": "string",
      "account": "integer",
      "time": "string",
      "region": "string",
      "resources": [],
      "detail": {
        "instance-id": "string",
        "state": "string"
      }
    },
    "parser": "json"
  }

Inspec
~~~~~~

Example Log::

  {
    "version": "1.4.1",
    "profiles": [
      {
        "supports": [],
        "controls": [
          {
            "title": null,
            "desc": null,
            "impact": 0.5,
            "refs": [],
            "tags": {},
            "code": "code-snip",
            "source_location": {
              "ref": "/lib/inspec/control_eval_context.rb",
              "line": 87
            },
            "id": "(generated from osquery.rb:6 de0aa7d2405c27dfaf34a56e2aa67842)",
            "results": [
              {
                "status": "passed",
                "code_desc": "File /var/osquery/osquery.conf should be file",
                "run_time": 0.001332,
                "start_time": "2017-01-01 00:00:00 -0700"
              },
              {
                "status": "passed",
                "code_desc": "File /var/osquery/osquery.conf should be owned by \"root\"",
                "run_time": 0.02764,
                "start_time": "2017-01-01 00:00:00 -0700"
              },
              {
                "status": "passed",
                "code_desc": "File /var/osquery/osquery.conf should be grouped into \"wheel\"",
                "run_time": 0.000425,
                "start_time": "2017-01-01 00:00:00 -0700"
              },
              {
                "status": "failed",
                "code_desc": "File /var/osquery/osquery.conf mode should cmp == \"0400\"",
                "run_time": 0.010417,
                "start_time": "2017-01-01 00:00:00 -0700",
                "message": "\nexpected: \"0400\"\n     got: \"0640\"\n\n(compared using `cmp` matcher)\n"
              }
            ]
          }
        ],
        "groups": [
          {
            "title": null,
            "controls": [
              "(generated from osquery.rb:1 813971f93b6f1a66e85f6541d49bbba5)",
              "(generated from osquery.rb:6 de0aa7d2405c27dfaf34a56e2aa67842)"
            ],
            "id": "osquery.rb"
          }
        ],
        "attributes": []
      }
    ],
    "other_checks": [],
    "statistics": {
      "duration": 0.041876
    }
  }

Schema::

  "inspec": {
    "schema": {
      "title": "string",
      "desc": "string",
      "impact": "float",
      "refs": [],
      "tags": {},
      "code": "string",
      "id": "string",
      "source_location": {
        "ref": "string",
        "line": "integer"
      },
      "results": []
    },
    "parser": "json",
    "configuration": {
      "json_path": "profiles[*].controls[*]"
    }
  }

Box.com
~~~~~~~

Example Log::

  {
    "source": {
      "item_type": "file",
      "item_id": "111111111111",
      "item_name": "my-file.pdf",
      "parent": {
        "type": "folder",
        "name": "Files",
        "id": "22222222222"
      }
    },
    "created_by": {
      "type": "user",
      "id": "111111111",
      "name": "User Name",
      "login": "user.name@domain.com"
    },
    "created_at": "2017-01-01T00:00:00-07:00",
    "event_id": "111ccc11-7777-4444-aaaa-dddddddddddddd",
    "event_type": "EDIT",
    "ip_address": "127.0.0.1",
    "type": "event",
    "session_id": null,
    "additional_details": {
      "shared_link_id": "sadfjaksfd981348fkdqwjwelasd9f8",
      "size": 14212335,
      "ekm_id": "111ccc11-7777-4444-aaaa-dddddddddd",
      "version_id": "111111111111",
      "service_id": "5555",
      "service_name": "Box Sync for Mac"
    }
  }

Schema::

  "box": {
    "schema": {
      "source": {
        "item_type": "string",
        "item_id": "integer",
        "item_name": "string",
        "parent": {
          "type": "string",
          "name": "string",
          "id": "integer"
        }
      },
      "created_by": {
        "type": "string",
        "id": "integer",
        "name": "string",
        "login": "string"
      },
      "created_at": "string",
      "event_id": "string",
      "event_type": "string",
      "ip_address": "string",
      "type": "string",
      "session_id": "string",
      "additional_details": {}
    },
    "parser": "json"
  },

Cloudwatch VPC Flow Logs
~~~~~~~~~~~~~~~~~~~~~~~~

AWS VPC Flow Logs can be delivered to StreamAlert via CloudWatch.

As they are compressed with deflate, we can use the special ``gzip-json`` for parsing and analysis.

CloudWatch logs are delivered as a nested record, so we will need to pass ``configuration`` options to the parser to find the nested records::

  "cloudwatch:flow_logs": {
    "schema": {
      "protocol": "integer",
      "source": "string",
      "destination": "string",
      "srcport": "integer",
      "destport": "integer",
      "action": "string",
      "packets": "integer",
      "bytes": "integer",
      "windowstart": "integer",
      "windowend": "integer",
      "version": "integer",
      "eni": "string",
      "account": "integer",
      "flowlogstatus": "string"
    },
    "parser": "gzip-json",
    "configuration": {
      "json_path": "logEvents[*].extractedFields",
      "envelope_key": {
        "logGroup": "string",
        "logStream": "string",
        "owner": "integer"
      }
    }
  }

osquery
~~~~~~~

osquery's schema depends on the table being queried and there are 50+ tables

**Option 1**: Define a schema for each table used::

  "osquery:etc_hosts": {
    "parser": "json",
    "schema": {
      "name": "string",
      ...
      "columns": {
        "address": "string",
        "hostnames": "string"
      },
      "action": "string",
      ...
    }
  },
  "osquery:listening_ports": {
    "parser": "json",
    "schema": {
      "name": "string",
      ...
      "columns": {
        "pid": "integer",
        "port": "integer",
        "protocol": "integer",
        ...
      },
      "action": "string",
      ...
    }
  },
  ...

This promotes Rule safety, but requires additional time to define the schemas


**Option 2**: Define a "loose" schema that captures all tables::

  "osquery": {
    "parser": "json",
    "schema": {
      "name": "string",
      "hostIdentifier": "string",
      "calendarTime": "string",
      "unixTime": "integer",
      "columns": {},                 # {} = any keys
      "action": "string"
    }
  },

.. warning:: In Option 2, the schema definition is flexible, but Rule safety is lost because you'll need to use defensive programming when accessing and analyzing fields in `columns`. The use of `req_subkeys` will be advised, see Rules for more details


CSV Examples
-----------

Example schema::

  "example_csv_log_type": {
    "parser": "csv",          # define the parser as CSV
    "schema": {
      "time": "integer",      # columns are represented as keys; ordering is strict
      "user": "string",
      "message": "string"
    },
    "hints": {                # hints are used to aid in data classification
      "user": [
        "john_adams"          # user must be john_adams
      ],
      "message": [            # message must be "apple*" OR "*orange"
        "apple*",
        "*orange"
      ]
    }
  },

For CSV, ``hints`` are used to aid in data classification since StreamAlert is stateless and does not have access to the CSV header


Example logs::

  1485729127,john_adams,apple            # match: yes (john_adams, apple*)
  1485729127,john_adams,apple tree       # match: yes (john_adams, apple*)
  1485729127,john_adams,fuji apple       # match: no
  1485729127,john_adams,orange           # match: yes (john_adams, *orange)
  1485729127,john_adams,bright orange    # match: yes (john_adams, *orange)
  1485729127,chris_doey,bright orange    # match: no



CSV Example w/nesting
---------------------

Some CSV logs have nested fields

Example logs::

  1485729127,john_adams,memcache us-east1    # time,user,message; message = role,region
  1485729127,john_adams,mysqldb us-west1


You can support this with a schema like the following::

  "example_csv_with_nesting": {
    "parser": "csv",
    "schema": {
      "time": "integer",
      "user": "string",
      "message": {
        "role": "string",
        "region": "string"
      }
    },
    "hints": [
      ...
    ]
  },


Key-Value (KV) Example
----------------------

Example schema::

  "example_auditd": {
    "parser": "kv",          # define the parser as kv
    "delimiter": " ",        # define the delimiter
    "separator": "=",        # define the separator
    "schema": {
      "type": "string",
      "msg": "string",
      "arch": "string",
      "syscall": "string",
      "success": "string",
      "exit": "string",
      "a0": "string",
      "a1": "string",
      "a2": "string",
      "a3": "string",
      "items": "string",
      "ppid": "integer",
      "pid": "integer",
      "auid": "integer",
      "uid": "integer",
      "gid": "integer",
      "euid": "integer",
      "suid": "integer",
      "fsuid": "integer",
      "egid": "integer",
      "sgid": "integer",
      "fsgid": "integer",
      "tty": "string",
      "ses": "string",
      "comm": "string",
      "exe": "string",
      "subj": "string",
      "key": "string",
      "type_2": "string",
      "msg_2": "string",
      "cwd": "string",
      "type_3": "string",
      "msg_3": "string",
      "item": "string",
      "name": "string",
      "inode": "string",
      "dev": "string",
      "mode": "integer",
      "ouid": "integer",
      "ogid": "integer",
      "rdev": "string",
      "obj": "string"
    }
  },


Syslog Example
--------------

Example schema::

  "example_syslog": {
    "parser": "syslog",
    "schema": {
      "timestamp": "string",
      "host": "string",
      "application": "string",
      "message": "string"
    }
  }


StreamAlert is configured to match syslog events with the following format::

  timestamp(Month DD HH:MM:SS) host application: message

Example(s)::

  Jan 10 19:35:33 vagrant-ubuntu-trusty-64 sudo: session opened for root
  Jan 10 19:35:13 vagrant-ubuntu-precise-32 ssh[13941]: login for jack

