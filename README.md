# tap-gsheets
This is a [Singer](https://singer.io) tap that extracts data from Google Spreadshits. It produces JSON-formatted data following the [Singer
spec](https://github.com/singer-io/getting-started/blob/master/SPEC.md).

# Configuration
The tap uses the Google API underneath for accessing sheets data, so you need to
[create some credentials accordingly](https://towardsdatascience.com/accessing-google-spreadsheet-data-using-python-90a5bc214fd2).

Once you have your credentials, you need to put them in the `gsheets_api` key in the configuration file so they are picked up by the tap. With another key `sheet_name` one selects the spreadsheet to be extracted.

The tap supports configuration files in JSON as much as in [HOCON](https://github.com/chimpler/pyhocon) format. Use a `.json` extension for JSON format and a `.conf` extension for HOCON format.

# Example
Tapping onto a Google sheet named `Investor Loans` can be achieved with a HOCO config file such as
```hocon
{ # config.conf
  sheet_name = "Investor Loans",
  gsheets_api {
    type = "service_account",
    project_id = "pmt-spv",
    private_key_id = ${PRIVATE_KEY_ID},
    private_key = ${PRIVATE_KEY},
    client_email = ${CLIENT_EMAIL},
    client_id = ${CLIENT_ID},
    auth_uri =  "https://accounts.google.com/o/oauth2/auth",
    token_uri = "https://oauth2.googleapis.com/token",
    auth_provider_x509_cert_url = "https://www.googleapis.com/oauth2/v1/certs",
    client_x509_cert_url = ${CERT_URL}
  }
}
```
Which will output something like
```bash
tap-gsheets -c config.conf
# {"type": "RECORD", "stream": "Investor Loans", "record": {"id": 11, "start_date": "2018-08-22", "end_date": "2020-04-17", "investor": "Banking Corp", "amount": 20000000, "interest_rate": 0.20, "add_to_capital": "FALSE", "user_id": "some_user", "created_at": "2018-08-22 11:00:40", "updated_at": "2018-08-22 11:00:40"}}
```

## Additional config
Import your service account config from the file Google provides:
```hocon
{ # config.conf
  gsheets_api {
    include "gcloud-team-name-001122334455.json"
  }
}
```

Change the table name format to singular form and convert column names to the valid database format:
```hocon
{ # config.conf
  underscore_columns = True,
  singular_table_name = True,
}
```

Process several sheets and worksheets in that sheets at a time:
```hocon
{ # config.conf
  sheets = [
    {
      name = "Investor Loans",
      worksheets = [
        PageA,
        PageB,
        PageC
      ]
    },
    {
      # ...
    }
  ]
}
```

By default date field define as date with "%Y-%m-%d %H:%M:%S" format. If there is necessary to be changed need to set specific_date_format:

```hocon
{ # config.conf
  sheets = [
    {
      name = "Investor Loans",
      worksheets = [
        PageA,
        PageB,
        PageC
      ],
    specific_date_format = "%Y-%m-%d" 
    },
    {
      # ...
    }
  ]
}
```

By default date field define as date. If there is necessary to be changed need to set date_processing to False:

```hocon
{ # config.conf
  sheets = [
    {
      name = "Investor Loans",
      worksheets = [
        PageA,
        PageB,
        PageC
      ],
    date_processing = False
    },
    {
      # ...
    }
  ]
}
```

Specify the row number (1-based) to start processing from, in case you want to skip some unnecessary rows. The default number is 1.

```hocon
{ # config.conf
  sheets = [
    {
      name = "Investor Loans",
      start_from_row = 5,
      worksheets = [
        PageA,
        PageB,
        PageC
      ]
    },
    {
      # ...
    }
  ]
}
```

# Overriding configuration
The configurations in the file can be overriden with the command line parameter `--overrides`,
which takes configuration overrides in a JSON string and applies them over the passed
config file.
```bash
tap-gsheets -c config.conf --overrides '{"sheet_name":"2019 Baseball Games"}'
```

# Install
To install the `tap-gsheets` utility as a system command, create and activate a
Python3 virtual environment
```bash
$ cd tap-gsheets
$ python3 -m venv ~/.virtualenvs/tap-gsheets
$ source ~/.virtualenvs/tap-gsheets/bin/activate
```
and install the package
```bash
$ pip install -e .
```
