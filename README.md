### This fork is designed to continue the work that was previously dependent on Censys by collecting similar data on a daily basis. The objective is to maintain the continuity of data acquisition, ensuring that researchers, analysts, and other stakeholders continue to have access to up-to-date information.

# K8s Censys information

This is a repository containing daily information about publicly visible Kubernetes clusters taken from [Censys](https://censys.io/) results, which can be useful for undestanding the uptake of different Kubernetes versions and platforms in the wild.

For background about why it's possible to gather this information, you can see [this blog post](https://raesene.github.io/blog/2021/06/05/A-Census-of-Kubernetes-Clusters/), [this blog post](https://raesene.github.io/blog/2022/07/03/lets-talk-about-kubernetes-on-the-internet/), and [this blog post](https://raesene.github.io/blog/2024/02/17/a-final-kubernetes-censys/)

The dataset runs from September 2022 to ~~5th February 2024~~. The data is in `/data/[YEAR]`

Censys have now (as of 17th Feb 2024) cancelled their free API access, so there will be no further updates to this dataset.

## "Data Pipeline"

It's important to undertstand how the data is sourced and gathered to understand limitations on what's available. The initial data comes from scraping the `/version` endpoint on exposed Kubernetes nodes. This is available by default in Kubernetes and in many Kubernetes distributions. A notable exception is AKS which does not make this available unauthenticated, so there are no stats related to AKS versions in this dataset.

The `k8s-censys.rb` script ran daily to pull information from the Censys API. First it pulled the data from Censys with a single API call, it then wrote that to a file called `[DATE]-k8s-version-info.json`.

The script then goes through the returned data and tries to assign it to a distribution based on strings in the version field. Many major distributions customise the k8s version number to include the product name, making this possible (the set of matches that look like `776c994` are for OpenShift which places a hex string in its version numbers) 

This data is written to a file called `[DATE]-k8s-version-info.csv` with three columns: `distribution`, `version`, and `count`. The `distribution` column is the name of the distribution, the `version` column is the version number, and the `count` column is the number of nodes that match that distribution and version.

The script then tries to extract the Kubernetes version from the version column (e.g. v1.23 from a string like `v1.23.14-gke.1800`) to allow for prevalance of specific k8s versions to be analysed. It writes this to a CSV file called `[DATE]-pure-versions.csv` with three columns: `major version`, `specific version` and `count` (e.g. `v1.24,v1.24.9,205208`)


## aggregate-versions.rb

There is a second script in the repository called `aggregate-versions.rb` this takes the data from the various "pure version" CSV files and creates a CSV that be imported into a spreadsheet package for analysis. It should output the major versions and the count for each day for each version.


