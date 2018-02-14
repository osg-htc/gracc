# GRACC Reporting

gracc-reporting is a python package (python 2.7+) that consists of a standard set of reporting libraries, and individual report executables that query GRACC, aggregate the data, and send reports to various stakeholders.

A lot of this information has been pulled from (and updated since then) [this presentation](https://docs.google.com/presentation/d/1FPfBx8lmHGYwaM0RTrlZ1ANc2Osx1G_Utcs0o3xKwOI/edit?usp=sharing).

## Reports in gracc-reporting

There are currently three subpackages in gracc-reporting.  In the future, we plan to split these out from the main set of reporting libraries.  These packages are:

* OSG Reports
    * OSG Flocking Report (osgflockingreport) - VO usage of OSG sites reported by flocking probes
    * OSG Project Usage Report (osgreport) - Project usage of OSG resources (OSG-Direct, OSG-Connect, XD)
    * OSG Missing Projects from Records Report (osgmissingprojects)
    * OSG VO Usage Per Site Report (osgpersitereport)
    * Top [N] Providers of Opportunistic Hours on the OSG (News Report) (osgtopoppusagereport)
    * Gratia Probes that haven't reported in the past two days (osgprobereport)
* FIFE Reports
    * Job Success Rate Report (jobsuccessratereport)
    * User Efficiency Report (efficiencyreport)
    * Top Wasted Hours by Users by VO Report (topwastedhoursvoreport)
* Minerva Report (deprecated)

### Source
The source code for gracc-reporting is on the [OSG's Github Page](https://github.com/opensciencegrid/gracc-reporting).  The Dockerfiles are contained in that repository as well, but the deployment structure using docker is detailed on the [Services](../ops/services.md#configuring-gracc-reporting) page.

## gracc-reporting structure

### General Report Structure
* Each report is subclass of ReportUtils.Reporter
* Implemented methods in most reports:
    * query: Define Elasticsearch query
    * ReportUtils.run_query:  Run that query and check for errors before proceeding
    * generate: Take results from query and parse them to generate raw data for any further processing
    * format_report (sometimes): Take processed data (sometimes from generate, sometimes from intermediate methods), put it into report format for TextUtils module to generate HTML, CSV, plain    text.
    * If this isn’t used, the formatting is done elsewhere in the report code.  For example, more complex HTML templates
    * send_report:  Email the report and check for errors
    * run_report: Run all of the above.  Mostly useful for more complex use cases (Probe Report, OSG Usage Per Site Report

### Running Reports in Development
* Each report has associated executable. 
* All can be used with -h, --help flag for full list of options
* Generally, something like: `osgflockingreport -s “2016-11-09 06:00:00” -e “2016-11-16 06:00:00”`
* Notes:
    * Almost all reports require a start time (-s) and end time (-e)
    * Times are assumed to be local time of host running reports.  The reports will convert to UTC to query GRACC.
    * Useful flags for all reports:
        * -d: dryrun (email only goes to test_to_emails in config file)
        * -n: no email sent (unless there’s an error.  Then emails go to dryrun folks)
        * -v: verbose

### Configuration files
Configuration files are simple toml files that are read through the toml python module.  There is one for each subpackage.  In keeping with setuptools' philosophy, the config files and html templates that are shipped with the source code are kept within the package at `src/graccreports/{config|html_templates}`.

#### Future deprecated behavior:
Currently, the common reporting libraries (in this case, ReportUtils.py) automatically look for the config file in the following places (in order):
* Override (see [below](#override-flags))
* `/etc/gracc-reporting/{config|html_templates}`
* Within package using pkg_resources

The plan is to change this in the future to look for the config file in an override, an environment variable, and then within the package.  `/etc/gracc-reporting` might be kept if the deployment model is changed.

Similar to config files, the reports automatically try to write logs to the following locations:
* Override
* Specified in config file
* `/var/log/gracc-reporting`
* `$HOME/gracc-reporting`
* `/tmp/gracc-reporting`

This will also be changed, so that the following order is observed:  override, environment variable, config file, `$HOME/gracc-reporting`.  In practice, the others are good, but unnecessary.

### Override flags:
* -c: Config file
* -T: HTML Template
* -L: logfile


## Installation
### Setuptools
* Clone repository: `git clone https://github.com/opensciencegrid/gracc-reporting`
* `pip install -r requirements.txt`
* `python setup.py install`

After this is complete, remember to change the config files within the package (or wherever you copy them) as needed (email recipients, etc.)

### RPM (future)
* RPM will be available online 
* Currently use this inside docker image
* Config files, HTML templates, etc. installed at `/etc/gracc-reporting/`
* If you’re running RHEL7, CentOS7, SL7, etc., [bug](https://bugzilla.redhat.com/show_bug.cgi?id=1450213) in package python-urllib3 in primary repo (RPM installation) - breaks python-elasticsearch 
* In my docker images, I uninstall python-urllib3, and pip install urllib3 to get around that (PyPI version is fine)


## Addendum:  More detailed developer notes

### Shared Libraries
* ReportUtils.py:  The big one
    * Reporter class:
        * Establish Elasticsearch client
        * Set correct ES index pattern (rather than just using `gracc.osg.raw-*`, unless we really want to query all raw records)
        * Run query with or without aggregations
        * Send simpler reports
        * Set up logging, get email info from config files, parse options to reports
    * Also has functions to find config, HTML template files, as well as handle errors
* IndexPattern.py:  Figures out index pattern given a date range and flags.  Assumes `gracc.osg.summary` if there's no config file `index_pattern` flag for a given report.
* TextUtils.py:  Used by ReportUtils to generate reports for simpler HTML templates
* TimeUtils.py:  Datetime validation and time zone handling (built heavily off of datetime and dateutil, creates timestamps for grafana)

### Building a new release
* Implemented methods in most reports:
* Edit setup.py if needed (if you have a new report, add executable, version update, etc.)
* `python setup.py sdist`
* Python package:
    * Copy out the tarball in dist/ wherever you want to deploy, or push it to github and pull from there
* RPM:
    * Spec file is in `config/` (symlink to `src/graccreports/config/`)
