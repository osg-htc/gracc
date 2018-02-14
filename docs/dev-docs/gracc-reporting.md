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

### Reports in gracc-reporting