This'll give you the 10 highest churn files (those with the most commits) found by `find`

`find {path} -name "*.rb | while read file; do echo $file`

`glo --$file | wc -l`;done | sort -k 2 - nr | head

[Attractor Rails](https://github.com/julianrubisch/attractor-rails) provides a web interface for assessing the quality of Rails projects in terms of churn and complexity