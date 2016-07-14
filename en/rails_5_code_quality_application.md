# Rspec Rails

Repo:
  `https://github.com/rspec/rspec-rails`
  
What to do:

remove the `test` folder

setup your gemfile
```
group :development, :test do
  gem 'rspec-rails'
end
```

run

`bundle install && rails generate rspec:install`

# Rubocop

Repo:
    `https://github.com/bbatsov/rubocop`
    
Setup your gemfile 

```
group :development, :test do
  gem 'rubocop', require: false
end
```

run

`rubocop --auto-gen-config`

It is going to generate a `.rubocop-todo.yml` file. Remove it and add the following content on it
```
# This is the configuration used to check the rubocop source code.
# Check out: https://github.com/bbatsov/rubocop

AllCops:
  TargetRubyVersion: 2.3

  Include:
    - '**/config.ru'
  Exclude:
    - 'vendor/**/*'
    - 'db/**/*'
    - 'db/schema.rb'
Rails:
  Enabled: true

Style/Documentation:
  Enabled: false
```

Now run `rubocop` and start to fix the issues, it is easier if you run `rubocop -a`.

# Rubocop-Rspec

Repo:
  `https://github.com/nevir/rubocop-rspec`
  
What to do:

Add the following line in your .rubocop.yml

`require: rubocop-rspec`

setup your gemfile
```
group :development, :test do
    gem 'rubocop-rspec', :require => false
end

```

# Rubycritic

Repo:
  `https://github.com/whitesmith/rubycritic`
  
What to do:

setup your gemfile
```
group :development, :test do
    gem 'rubocop-rspec'
end
```
run

`rubycritic`


# Brakeman

Repo:
  `https://github.com/presidentbeef/brakeman`
  
What to do:


setup your gemfile
```
group :development, :test do
    gem 'brakeman', :require => false
end
```
run

`brakeman`
