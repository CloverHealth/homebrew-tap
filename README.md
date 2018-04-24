# homebrew-tap

This is Clover Health's Homebrew Tap, which features formulas to help pin
Postgres to 9.6 and Postgis to 2.3.

Homebrew will pull in the latest version of formulas when they are upgraded,
meaning that users can inadvertently be upgraded to Postgresql 10. The
postgresql.rb formula here ensures that 9.6.8 is installed, and the postgis.rb
formulate ensures that 2.3.2 is installed.

## Installing Postgres 9.6 and Postgis 2.3

First ensure you have upgraded to the latest homebrew:

```sh
brew update
```

It's recommended to go ahead and clear any existing Postgres and Postgis
installations:

```sh
brew uninstall postgis --force
brew uninstall postgresql --force
brew cleanup
brew services stop postgresql
```

After this, install Clover's custom brew tap:

```sh
brew tap cloverhealth/homebrew-tap
```

Then install the latest version of Postgres and unlink it. This is done first so that
we can explicitly switch to an earlier version before installing Postgis:

```sh
brew install postgresql
brew unlink postgresql
```

The previous install of postgresql runs `initdb`, which creates database structures incompatible with 9.6.8. This needs to be removed with:

```sh
rm -rf /usr/local/var/postgres
```

Now install Postgres from this tap with:

```sh
brew install cloverhealth/tap/postgresql  # yes, without the homebrew-
```

This may appear to fail with the following error message:

```sh
$ brew install cloverhealth/tap/postgresql
==> Installing postgresql from cloverhealth/tap
==> Downloading https://ftp.postgresql.org/pub/source/v9.6.5/postgresql-9.6.5.tar.bz2
######################################################################## 100.0%
==> ./configure --prefix=/usr/local/Cellar/postgresql/9.6.5 --datadir=/usr/local/share/postgresql --libdir=/usr/
==> make
==> make install-world datadir=/usr/local/Cellar/postgresql/9.6.5/share/postgresql libdir=/usr/local/Cellar/post
Error: Failed to install plist file
==> /usr/local/Cellar/postgresql/9.6.5/bin/initdb /usr/local/var/postgres
Error: Calling <<-EOS.undent is disabled!
Use <<~EOS instead.
/usr/local/Homebrew/Library/Taps/cloverhealth/homebrew-tap/postgresql.rb:108:in `caveats'
Please report this to the cloverhealth/tap tap!
Or, even better, submit a PR to fix it!
```

You may ignore this error.

Now you will have both 9.6.8 and the latest version of Postgres installed.
Switch to 9.6.8 with:

```sh
brew switch postgresql 9.6.8
```

Postgis 2.3 can be installed with:

```sh
brew install cloverhealth/tap/postgis
```

Try running and accessing Postgres with the following:

```sh
brew services start postgresql
psql postgres  # It should show 9.6.8 as the version on the prompt
```

After running `psql postgres`, type the following in the prompt to verify your Postgis installation:

```sh
drop extension postgis;  -- This might fail if it wasn't previously installed
create extension postgis;  -- This should pass
select ST_Distance(
  ST_GeometryFromText('POINT(-118.4079 33.9434)', 4326), -- Los Angeles (LAX)
  ST_GeometryFromText('POINT(2.5559 49.0083)', 4326)     -- Paris (CDG)
);  -- This should print a row with 121.898285970107 as a value
```
