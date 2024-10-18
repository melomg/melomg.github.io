Welcome to my blog running on [melihgultekin.dev](https://melihgultekin.dev/).

Development
-----------

### Run the site locally
```bash
gem install bundler # first time only
bundle install # first time only
bundle exec jekyll serve
```

### Troubleshoot

To kill the process:

```bash
ps aux |grep jekyll |awk '{print $2}' | xargs kill -9
```

## License

```plaintext
Copyright [2024] [Melih Gültekin]

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```