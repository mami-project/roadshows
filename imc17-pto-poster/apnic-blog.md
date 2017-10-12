# An Observatory for Path Transparency Measurement

Deploying new protocols in the Internet is hard. 

Though the end-to-end principle and the four-layer TCP/IP architecture suggest
that what happens above the IPv4 or IPv6 header isn't any of the network's
business, the widespread deployment of firewalls, network address translators,
proxies, and other middleboxes at layers 4 and 7 mean that in practice, TCP
usually works, UDP usually works, and for everything else, your mileage may
vary.

The design and implementation of new protocols and protocol extensions, such as
Multipath TCP (MPTCP), must consider the potential actions of middleboxes in
modifying their headers or dropping their packets, with correspondingly complex
fallback code. These design decisions should be backed up by measurement,
though: while it makes sense to detect and respond to an impairment that happens
1% of the time, it makes less sense to do so for one that happens on one path in
ten million. In addition, as protocol extensions like MPTCP and Explicit
Congestion Notification (ECN) and new protocols such as QUIC see increased
deployment, quantifying and localizing the extent of middlebox-driven
impairments to these protocols is useful to the operations community in
debugging connectivity and performance problems linked to their use.

To this end, the [MAMI project](https://mami-project.eu) has designed the Path
Transparency Observatory (PTO), an open-source and publicly-accesible repository
for measurements of _path transparency_. A path through the Internet is said to
be transparent to a given protocol when packets using that protocol are received
without drops or modification at their destination. The PTO is designed to meet
the goals of _comparability_ and _repeatability_ in a way that is agnostic to
the measurement method for determining the presence of impairments to path
transparency. 

We learned some lessons from a [pilot
release](https://mami-project.eu/index.php/2017/03/01/path-transparency-observatory-pto-goes-live/)
of the PTO in March, which makes a limited dataset available at
[https://observatory.mami-project.eu](https://observatory.mami-project.eu)).
First among these is that for the types of measurements the PTO makes available,
which comprise hundreds of millions to tens of billions of observations, "big
data" approaches are overkill, adding runtime overhead and administrative
complexity for not much win. We therefore went back to the drawing board for a
"medium data", fully RESTful redesign of the PTO.

## Design

![PTO design](pto-design.png)

The PTO is implemented as a RESTful API around two data stores, as shown in the figure above:

- a _raw data_ store containing unprocessed measurement results received from
  measurement tools such as [PATHspider](https://pathspider.net).

- an _observation_ store containing processed measurement results in terms of a
  _condition_ observed on a _path_ during a given _time_ interval, which can be
  queried through a simple query interface.

We achieve _comparability_ through the unified data model in the observation
store: conditions are defined in terms of the _feature_ (e.g. ECN) and the
_aspect_ of that feature (e.g. connectivity impairment, ability to negotiate,
ability to signal) they measure. Each feature and aspect is associated with a
number of _states_: for example, the `ecn.connectivity` aspect has the states
`works` (trying to use ECN doesn't have any impact on connectivity), `broken`
(trying to use ECN makes a connection impossible, e.g. because ECN SYN packets
are dropped), or `offline` (no measurement was possible because the far endpoint
wasn't online). Paths are expressed in terms of IP addresses, prefix, or BGP
anonymous system numbers, allowing multiple resolutions of data depending on the
sensitivity of the raw address information.  Data in the raw store is normalized
into observations in terms of these conditions and paths, which makes queries
across hetereogeneous source data made by different measurement tools possible,
as long as they are measuring the same aspects of the same features. 

We achieve _repeatability_ by ensuring that each stage of a given result stores
its provenance, referring to the data it's based on in terms of stable links to
that data. The observation store is organized into _observation sets_, the
results of running a specific analysis on a specific raw data file or other
observation set. A query's metadata links to all the observation sets which
answer it; these sets' metadata link in turn to the observation sets and
analyses on which they are based, which eventually link to the raw data files
from which the observations were normalized. Metadata for each analysis includes
references to the specific commit of the analysis codebase which was used; given
access to the raw data and source code repository, every observation can
therefore be recreated from its antecedents. 

Normalization and analysis can happen on the server hosting the PTO itself, or
remotely, by retrieving raw data and observations and posting the resulting
observations to the observation store via the API. Since provenance is achieved
through URL links, raw data and derived observations can even be dispersed among
multiple instances of the PTO, allowing finer-grained control over possibly
sensitive raw data.

Public release of our new lightweight PTO will happen in the coming weeks. It is
based on an implementation of a RESTful API in bare Go (i.e., without any web
frameworks), with raw data storage backed by the filesystem, and observation
storage in the venerable PostgreSQL RDBMS.

## Results

As reported at [ANRW](https://irtf.org/anrw/2017/anrw17-final16.pdf) in Prague
in July of this year, the normalization and refinement of analysis of path
transparency measurements at scale offered by the PTO has already led to
interesting insights about the nature of path transparency in the Internet.
Impairments to the use of ECN, already found to be rare, appear to be dependent
on the path between the source and destination primarily in jurisdictions with
documented deployments of heterogeneous, TCP-intercepting Internet censorship
infrastructure. 