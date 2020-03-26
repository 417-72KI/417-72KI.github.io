

```shell
ruby -e 'require "yaml"; yaml = YAML.load_file(".gitlab-ci.yml"); variables = yaml["variables"]; variables.each { |key, value| cmd = "export #{key}=\"#{value}\""; system("echo #{cmd}"); }'
```
