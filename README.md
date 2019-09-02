# Google Summer of Code 2019, with PSF SubOrg: StarKit
<p align="center"> <img src="gsoc_logo.png" width="700" height="400" /> </p>

It was an incredible learning experience with GSoC and StarKit! I can't believe three months have passed so quick that I'm writing this final report, summarizing all the exciting work I did in these Summers (27 May - 26 Aug 2019).


## Overview
[StarKit Collaboration](https://github.com/starkit) provides Python packages for astronomers to determine the physical properties of stars. Currently it provides 2 packages - one is [StarKit](https://github.com/starkit/starkit) for fitting stars & analysing stellar spectra, and other is [WSynPhot](https://github.com/starkit/wsynphot) for doing synthetic photometry. During Summers, my work mainly revolved around WSynPhot package where I did various improvements to make it steady for use. With each problem I fixed, I learned a variety of things about package development. Thanks to my mentor [@wkerzendorf](https://github.com/wkerzendorf) whose guidance was pivotal for upgrading our packages.

Originally my proposed project: [Auto-generating Filter Curves & Photometry](https://summerofcode.withgoogle.com/projects/#5799260895838208), was aimed to develop analytical web interfaces that makes it easier for astronomers to access the filter curves and to calculate the photometry of Stars by interacting with data. But before I could add these new features to our package, I needed to resolve several problems present in our packages (specifically WSynPhot) since they were in a very initial phase of development. So for the most part of Summers (10 weeks), I worked on resolving those problems and improving several aspects of our packages. And finally in the last 2 weeks, I got to work on developing a prototypical web interface for interactively plotting star's spectrum!


## What has been done?
During this 3-month period, I got **30+ PRs merged** along with some direct commits to our repositories. You can find my weekly progress at [my python-gsoc blog](https://blogs.python-gsoc.org/en/jaladh-singhals-blog/) wherein I've also shared various things I learned each week. In this report, I have summarized my work as following deliverables:

### 1. Setup of CI/CD Azure pipelines for both packages
Initially we were using Travis for CI(Continuous Integration) and RTD(Readthedocs) for deploying our docs, then we decided to deploy our docs to GH-pages and hence used Doctr. But then we found Azure pipelines to be more promising in terms of its customizability, so we ended up creating both CI and docs CD pipeline on Azure.
- Setup Testing (CI) Pipeline for starkit - [#56](https://github.com/starkit/starkit/pull/56)
- Setup CD (Continuous Deployment) pipeline for deploying starkit docs to gh-pages - [50f7167](https://github.com/starkit/starkit/commit/50f71671710101e2128f0c210a4fd92d80faf647)
- Setup Testing (CI) Pipeline for wsynphot - [e76b26f](https://github.com/starkit/wsynphot/commit/e76b26ffba78fab65c47d322cf77f2e97f60a344)
- Setup CD pipeline for deploying wsynphot docs to gh-pages - [e4b0940](https://github.com/starkit/wsynphot/commit/e4b09404186f948b9768b0f939f53e07bd5b4f22)
- Removed Travis & RTD files after setting up Azure for starkit - [#58](https://github.com/starkit/starkit/pull/58), [#60](https://github.com/starkit/starkit/pull/60)
- Removed Travis & RTD files for wsynphot - [#24](https://github.com/starkit/wsynphot/pull/24), [#26](https://github.com/starkit/wsynphot/pull/26)
- Helped a fellow GSoC student at our sister org [TARDIS](https://github.com/tardis-sn) in setting up docs CD pipeline and writing documentation for the same - [#939](https://github.com/tardis-sn/tardis/pull/939)
- Made docs CD pipeline to execute the notebook when building docs, for which I made cached filter data available for execution by using Azure artifacts - [#40](https://github.com/starkit/wsynphot/pull/40)


### 2. Remodelling of the data access mechanism of WSynPhot
While imaging stars, astronomical filters are used with telescopes to capture specific wavelengths of starlight. Wsynphot depends on this filter data (which we obtain from [SVO FPS](http://svo2.cab.inta-csic.es/theory/fps/)), for doing photometric calculations. There's an interesting story behind how I get into this work which was not even planned. It all started with resolving an issue of unclean data and we ended up with having dedicated I/O modules for getting & caching filter data from SVO. We tried a lot of things and kept switching to better solutions in the process, so I also have some closed PRs for the work we dropped.

Initially wsynphot accessed the filter data, by scraping it from SVO web interface and ingesting it in required format in an HDF file, which was then uploaded on a server. Since SVO keeps on updating, we decided to design a scheduled pipeline to auto ingest the updated filter data HDF file. While developing this, we found that there exists a VO interface (HTTP queries based API) for SVO which can be used to directly fetch data from SVO without ingesting it in HDF. Thus we dropped earlier code and I wrote a module to get data from SVO in real-time. But since it has a time & connectivity limitation, so we decided to create another module to cache the filter data as VOTables on user's disk. And I also developed a mechanism to update the cached data so that user have access to up-to-date filter data.
- Fixed the unclean filter data (garbage strings & empty dataset problems) - [#13](https://github.com/starkit/wsynphot/issues/13), [#18](https://github.com/starkit/wsynphot/pull/18), [#22](https://github.com/starkit/wsynphot/pull/22) [closed]
- Created a pipeline to auto-ingest filter data which executes ingestion notebook, saves error logs and deploys the generated HDF on a server - [#25](https://github.com/starkit/wsynphot/pull/25) [closed - code is no longer available because I made a terrible mistake of creating PR from my fork's master, a learning I'll always remember]
- Developed **get_filter_data module** to get data directly from SVO using API, along with unit tests and documentation for its use - [#27](https://github.com/starkit/wsynphot/pull/27), [#29](https://github.com/starkit/wsynphot/pull/29)
- Contributed above module as SVO FPS query tool to Astroquery (a famous package to access online astronomical resources) - [#1495](https://github.com/astropy/astroquery/issues/1495), [#1498](https://github.com/astropy/astroquery/pull/1498) [open - under their next milestone]
- Developed **cache_filters module** to cache filter data on user's disk (download/read data), along with unit tests and documentation for its use - [#30](https://github.com/starkit/wsynphot/pull/30), [#31](https://github.com/starkit/wsynphot/pull/31), [#32](https://github.com/starkit/wsynphot/pull/32), [#35](https://github.com/starkit/wsynphot/pull/35)
- Integrated cache_filters module in base module of wsynphot and removed HDF handling code - [#36](https://github.com/starkit/wsynphot/pull/36)
- Setup a mechanism to update cached filter data which automatically reminds the user to update their cache if it becomes >1 month since last update - [#37](https://github.com/starkit/wsynphot/pull/37)
- Improved unit tests of both developed modules extensively by using mock testing, to reduce their execution time and to increase their code coverage - [#39](https://github.com/starkit/wsynphot/pull/39)
- Reported several errors I recognised in filter data at SVO FPS by directly mailing their support, and got them fixed


### 3. Other fixes/improvements
- Reported CI build failure caused by pluggy v0.12 update, whose maintainers fixed the importlib_metadata conda package that was causing the problem - [#220](https://github.com/pytest-dev/pluggy/issues/220)
- Removed python 2.7 support from both of our packages, now we only support python 3 - [#62](https://github.com/starkit/starkit/pull/62), [#28](https://github.com/starkit/wsynphot/pull/28)
- Disabled use_2to3 flag in both the packages to make development mode of setup.py work properly in python3 - [#63](https://github.com/starkit/starkit/pull/63), [#38](https://github.com/starkit/wsynphot/pull/38)
- Created redirects at RTD stating docs moved to gh-pages, for both of our packages - [64bde3c](https://github.com/starkit/starkit/commit/64bde3c225f4fec3c78c7edcc2ea349b5e57c063)...[f0fd026](https://github.com/starkit/starkit/commit/f0fd026f9e4012eeaa880b51ae0abba2dddcf4d7), [6e83e33](https://github.com/starkit/wsynphot/commit/6e83e337a451738c4b2ba896cf3d06b29333e206)...[6b3e463](https://github.com/starkit/wsynphot/commit/6b3e463983a077529d6686fd8dec217af71c9071) and also for [tardis](https://github.com/tardis-sn/tardis) & [carsus](https://github.com/tardis-sn/carsus) packages of our sister org - [#947](https://github.com/tardis-sn/tardis/pull/947), [#136](https://github.com/tardis-sn/carsus/pull/136), [#138](https://github.com/tardis-sn/carsus/pull/138), [#139](https://github.com/tardis-sn/carsus/pull/139). Later on I found a better solution to implicitly redirect RTD docs URL to our gh-page URL, which I setup for all 4 of these packages from their RTD account directly.
- Fixed problem in accessing calibration files in wsynphot when doing photometric calculations - [#33](https://github.com/starkit/wsynphot/pull/33)
- Fixed failing I/O tests at wsynphot due to a major update in filter data at SVO - [#34](https://github.com/starkit/wsynphot/pull/34)
- Added a page in starkit docs listing all the test grids we have made available for analysing the spectra; also fixed some problems in main TOC - [#64](https://github.com/starkit/starkit/pull/64)
- Updated gitignore of starkit to ignore more no. of file types - [#66](https://github.com/starkit/starkit/pull/66)


### 4. Development of interactive Interfaces
Since we were done with most fixes so in last 2 weeks, I started exploring and experimenting with several tools that can create interactive web interfaces from Jupyter Notebooks.
- Developed a [prototypical web app](https://mybinder.org/v2/gh/jaladh-singhal/starkit/binder?urlpath=voila%2Frender%2Finterfaces%2Finteractive_spectrum.ipynb) to interactively plot the spectrum of star using voila & mybinder - [35c6a26](https://github.com/jaladh-singhal/starkit/commit/35c6a2616c014785a410dcef09371e609e00d474)...[433bebd](https://github.com/jaladh-singhal/starkit/commit/433bebd99eff1bc10103e5b892eeac44dea7385c)
- Added an example notebook in starkit docs to interactively plot spectrum of star, so that users can use it locally - [#65](https://github.com/starkit/starkit/pull/65)


### TL;DR
All of my work during GSoC (and even pre-GSoC) period can be accessed from [this complete list](https://github.com/pulls?page=2&q=author%3Ajaladh-singhal+created%3A%3C2019-08-26+user%3Astarkit+user%3Atardis-sn+user%3Aastropy+user%3Apytest-dev&utf8=%E2%9C%93) of issues & PRs I created.

## What is left to do?
Yet GSoC has ended but my contributions to StarKit won't end! I've planned to develop interactive Photometry interface (part of my proposal) post GSoC. Since no such interface exists yet, therefore I am eager to contribute it to Astronomy community.

Besides it, there are a few more things that can be done:
- Integrate auto-generable filter curves & list in wsynphot docs - [#23](https://github.com/starkit/wsynphot/pull/23) [open PR]
- Add docs passing badge to both packages - [#41](https://github.com/starkit/wsynphot/pull/41) [open PR]
- Fix the problem of docs CD pipeline failing for PR builds
- Resolve the cause of warnings given by Astropy votable parser
- Fix colored logger to make it work for nested function calls


## Conclusion
Summers 2019 were definitely very pleasant and enjoyable for me. I am thankful to Google for organising such a great program that introduced me to FOSS world and increased my knowledge manifold. A big thanks to my mentors at StarKit for letting me become a part of StarKit, and for helping me in improving not only our packages but also myself as a developer. And lastly thanks to all FOSS communities for helping each other and for letting novice developers like me, escalate their skills by working on their cool projects!

**_Signing off with :heart: to Open Source!_**
