A major trend in the startups is the creation of projects in a astronomical speed so that they can be placed in the market to be validated . This being done their destination is defined between a range of options that can be generalized in:

**It needs maintenance or not.**

A problem in this approach is the quality of the project in general, mostly on code quality and security.

Once the founder of [LinkedIn](www.linkedin.com) [Reid Hoffman](https://www.linkedin.com/in/reidhoffman) said:

*If you are not embarrassed by the first version of your product, you’ve launched too late.*

More about this philosophy [here](http://www.businessinsider.com/the-iterate-fast-and-release-often-philosophy-of-entrepreneurship-2009-11)

As it applies to what the consumer will find, it should not be applied to the code. It is clear that we should not take care of large customizations, but there are small practices that *can* ensure a good maintenance as well as saving time (our most expensive resource) and money.

Here at [Codelitt](codelitt.com) we have a lot of projects built with Ruby/Rails, take a look [here](https://www.quora.com/Why-do-so-many-startups-use-Ruby-on-Rails) if you want to know why. As we work mostly with startups projects we’ve decided to define a set of gems that should be in every project in order to help us to maintain our best practices.

#### Code quality

As quality is something too abstract we’ve decided to focus in three main points:

 1. Readability
 2. Organization
 3. Maintainability

Our code must be **readable** in a way that the the developer doesn’t waste too much time to understand what it does.

Our code must be well **organized** in packages/modules/classes. We strive to follow the rules of the [Rails Style Guide](http://guides.rubyonrails.org/index.html).

Our code must be **maintainable**. Although we work with startups, we do care about the future of each one, so if in the future our client decides to improve his product, a lot of money and a lot of his time (and our time) will be saved.

##### [Rubocop](https://github.com/bbatsov/rubocop)

RuboCop is a Ruby static code analyzer, it searches for smells and bad practices defined by the community.
Examples of smells are:

 1. Readability
 2. Organization
 3. Maintainability

The official good practices sources are [Ruby style guide](https://github.com/bbatsov/ruby-style-guide) and [Rails style guide](https://github.com/bbatsov/rails-style-guide)

It has a huge range of good practices and allows us to customize them, like in the example below:

```
Rails:
 Enabled: true

Metrics/LineLength:
 Max: 120

Style/Documentation:
 Enabled: false

Style/BlockDelimiters:
 Enabled: false
```

Besides showing the code smells, it offers a great auto fix system that when used with rails it does a really good update in the code, like turning:

```ruby
def my_attribute
 my_attribute
end
```

into

``` ruby
 attr_accessor :my_attribute
```

As soon as I found it I applied it in a huge project that was already in production. It went from 1400 warnings to 214, so as you may see it fixed more than 80% of the code smells, mostly of the non fixed ones were too complex, some of them even I had some problems to fix.

Alongside with [Rubocop](https://github.com/bbatsov/rubocop) we use [Rubocop Rspec](https://github.com/nevir/rubocop-rspec) because it has a lot of RSpec specific good practices.

Here at [Codelitt](codelitt.com) we use it to guarantee good practices in our code, as sometimes even developers with years of experience make small mistakes like the wrong use of indention, commas, variables declared but not used and so it goes.
We have a policy of 0 warnings in our CI, before the build we run rubocop and in case of a warning being raised the build fails.

##### [Rubycritic](https://github.com/whitesmith/rubycritic)

Rubycritic is another static code analyzer, it wraps around the static analysis gems [Reek](https://github.com/troessner/reek), [Flay](https://github.com/seattlerb/flay) and [Flog](https://github.com/seattlerb/flog).We use mainly this gem because it offers an overview of the duplicated code and complexity, as you can see in the report below:

![alt text](http://www.clipular.com/c/5227312822353920.png?k=xKPmaAjaIBnIg-ZwOJoLbZVlQZ8 “Ruby Critic report example”)

Here at [Codelitt](codelitt.com) we care a lot about the [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) principle and the report that this gem offers is really helpful.

#### Security

As we work with projects that go from simple sites to money transfer systems, by default we do care about security in all of them.

Besides the server good practices and a special care to sensitive data we obviously can’t allow to ignore the harm that may be done in our applications by attacks with: DDOS, SQL Injection and so on. To ensure that our applications are minimally secure we use the following gems:

###### [Brakeman](https://github.com/presidentbeef/brakeman)

Brakeman is a gem that checks Ruby on Rails applications for security vulnerabilities, it raises warnings for each one that is found.

Here at [Codelitt](codelitt.com) our policy is 0 security warnings. As it is the first step in our build process, if it finds any security gap it fails and don’t build the project.

###### [Dawnscanner](https://github.com/thesp0nge/dawnscanner)

Dawnscanner is a source code scanner designed to review ruby code for security issues. We use it alongside with [Brakeman](https://github.com/presidentbeef/brakeman) to ensure a redundancy in our security verification. It runs after [Brakeman](https://github.com/presidentbeef/brakeman) and fails if it finds any gap as well.

###### [Bundler-audit](https://github.com/rubysec/bundler-audit)

Bundler Audit checks the Gemfile.lock and searches for security gaps reported in each gem used in the project. If it finds any problem in the version of a gem, it searches for one that is secure and asks us to update it. If a secure one doesn’t exist, well, it is time to search for another.

While we focus on the general security of our applications it is common to forget that the product is a combination of gems and code that we write, so if any gem used in the project has a security gap then it is the enemy inside that we have.

Here at [Codelitt](codelitt.com) the verification with [Bundler-audit](https://github.com/rubysec/bundler-audit) is the third step before the build. When it finds a compromised gem we search for a safe version, in the case of not finding one, we just throw it away and search for another way to solve the problem.

#### Code coverage

Code coverage is important because with this data we can ensure that our application is being minimally tested. As code lines covereded don't guarantee that the critical points of the application are being tested (which needs to be our main focus) we can’t use it as basis to make sure that our applications are safe, but only that something that was working will keep working after some change in the code.

##### [Simplecov](https://github.com/colszowka/simplecov)

SimpleCov is a code coverage analysis tool for Ruby. Besides the offer of good test reports it allows us to configure our tests to fail if a coverage percentage is not met.

Here at [Codelitt](codelitt.com) we define that the minimum code coverage should be at least 90%.

#### Conclusion

It is a summary of the principles and gems that we use in our Ruby/Rails projects here at [Codelitt](codelitt.com). It will be updated every time that we find new gems or good practices. If you know a better solution or have some recommendation please let me know in the comments or by email. I would really appreciate it, mine is kaio@codelitt.com.

If you want to know more about our practices, take a look in our [repository](https://github.com/codelittinc/incubator-resources) it has a lot of cool stuff.
