README
======

record package
    => packages (list of URIs of all the release packages that were used to create this record package)
    => records (it's an array)
        => record
            => release-packages (it's the same info from the URIs from the 'packages')
                => release (can be):
                    => URI to the actual data
                        OR
                    => actula data from 'release.json' file
