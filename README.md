# <a name="title"></a> rbenvinator

> Building Ruby version tarballs for rbenv. Because your time is valuable.

## <a name="prerequisites"></a> Pre-Requisites

### <a name="prerequisites-vagrant"></a> Vagrant

You need to have an up-to-date version of [Vagrant](http://vagrantup.com/)
installed on your system.

### <a name="prerequisites-vagrant-base-boxes"></a> Vagrant Base Boxes

After Vagrant is installed you must add the base boxes you want to use. If
you need to create a base box then refer to the Vagrant documentation or the
[VeeWee](https://github.com/jedi4ever/veewee/) project for more details.

I also maintain a collection of
[VeeWee definitions](https://github.com/fnichol/veewee-definitions) that I use
with this project.

### <a name="prerequisites-librarian-gem"></a> Librarian Gem

The [Librarian](http://rubygems.org/gems/librarian) also needs to be installed
by running:

```bash
$ gem install librarian
```

### <a name="prerequisites-s3"></a> An AWS/S3 Account

rbenvinator will upload the tarball packages artifacts as they are built to
S3 for hosting. You will need:

* Your access key id
* Your secret access key
* An S3 bucket created for uploading

## <a name="setup"></a> Setup

First, clone down the project:

```bash
$ git clone git://github.com/fnichol/rbenvinator.git
$ cd rbenvinator
```

Next initialize your configuration with:

```bash
$ rake init
```

Now edit the `config.yml` and fill in your AWS/S3 credentials and enumerate
the Vagrant base box/Ruby version combinations you want to build against.

## <a name="build"></a> Build Them All!

Now run through each base box (one at a time) and compile all your Ruby
versions! Maybe this'll be the last time...

```bash
$ time (rake build)
```

## <a name="development"></a> Development

* Source hosted at [GitHub][repo]
* Report issues/questions/feature requests on [GitHub Issues][issues]

Pull requests are very welcome! Make sure your patches are well tested.
Ideally create a topic branch for every separate change you make. For
example:

1. Fork the repo
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Added some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## <a name="authors"></a> Authors

Created and maintained by [Fletcher Nichol][fnichol] (<fnichol@nichol.ca>)

## <a name="license"></a> License

MIT (see [LICENSE][license])

[license]:      https://github.com/fnichol/rbenvinator/blob/master/LICENSE
[fnichol]:      https://github.com/fnichol
[repo]:         https://github.com/fnichol/rbenvinator
[issues]:       https://github.com/fnichol/rbenvinator/issues
[contributors]: https://github.com/fnichol/rbenvinator/contributors
