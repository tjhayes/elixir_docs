# Required Software

    Unix (needed for ASDF. E.G. WSL on Windows)
    ASDF (will install Nodejs, Erlang and Elixir with this)

# Initial Setup

    mix new <appname> --umbrella 
    cd <appname>/apps 
    mix phx.new <appname> 
    mix phx.new.web <appname_web> 
    cd .. 
    git init 
    git commit -am "initial commit"



# Set Tool Versions with ASDF

    https://alchemist.camp/episodes/asdf-language-versions 
    cd <appname>

    asdf current 
    asdf plugin-add elixir -- add a new language 
    asdf list -- list all installed languages 
    asdf list-all elixir -- list all available  versions of a languages 
    asdf install elixir <version> 
    asdf global elixir <version>

    asdf install nodejs <version> 
    asdf install erlang <version> 
    asdf install elixir <version>

    bash ~/.asdf/plugins/nodejs/bin/import-release-team-keyring 
    asdf local nodejs 14.9.0 
    export KERL_CONFIGURE_OPTIONS="--disable-debug --without-javac" 
    asdf local erlang 23.0.2 
    asdf local elixir 1.10.4-otp-23 
    (elixir and erlang versions must match: 23 in this case)




# Package Setup

		(version numbers as of 2020-09-02. See https://hex.pm/packages/)
		 
		Umbrella: 
			  {:dialyxir, "~> 1.0", only: :test, runtime: false}, 
			  {:credo, "~> 1.4", only: [:dev, :test], runtime: false}, 
			  {:excoveralls, "~> 0.13.1", only: :test} 
		Web:
			  {:drab, "~> 0.10.5"},


		Misc commands: 
		mix deps.get          # Gets all out of date dependencies 
		mix deps.tree         # Prints the dependency tree 
		mix deps.unlock       # Unlocks the given dependencies 
		mix deps.update       # Updates the given dependencies

		run mix deps.get




# Setup Umbrella Mix.exs

## Add the following to project:
      test_coverage: [tool: ExCoveralls],
      dialyzer: [
        ignore_warnings: "dialyzer.ignore-warnings",
        list_unused_filters: true
      ],
      aliases: aliases()

## Create aliases:
	  defp aliases do 
		[ 
		  "my.format": [ 
			"format --check-formatted", 
			"credo --strict" 
		  ], 
		  "my.test": [ 
			"cmd MIX_ENV=test mix coveralls.html", 
			"cmd MIX_ENV=test mix dialyzer" 
		  ], 
		  "my.check": [ 
			"my.format", 
			"my.test" 
		  ], 
		  "my.phx": [
			"cmd npm run deploy --prefix ./assets", 
			"phx.digest.clean", 
			"phx.digest" 
		  ],
		  "my.release": [ 
			"my.check", 
			"deps.get --only prod", 
			"cmd MIX_ENV=prod mix compile", 
			"my.phx",
			"cmd MIX_ENV=prod mix release" 
		  ] 
		] 
	  end

# Set up ExCoverAlls

    create coveralls.json in root directory of umbrella project:

    {
      "skip_files": [
        "deps",
        "lib/<appname>/application.ex",
        "lib/<appname>_web/channels/user_socket.ex",
        "lib/<appname>_web/commanders",
        "lib/<appname>_web.ex",
        "lib/<appname>_web/gettext.ex",
        "lib/<appname>_web/views",
        "test"
      ],
      "coverage_options": {
        "minimum_coverage": 90,
        "treat_no_relevant_lines_as_covered": true
      }
    }

# Set up .gitignore in root

    # The directory Mix will write compiled artifacts to.
    /_build/

    # If you run "mix test --cover", coverage assets end up here.
    /cover/

    # The directory Mix downloads your dependencies sources to.
    /deps/

    # Where 3rd-party dependencies like ExDoc output generated docs.
    /doc/

    # Ignore .fetch files in case you like to edit your project deps locally.
    /.fetch

    # If the VM crashes, it generates a dump, let's ignore it too.
    erl_crash.dump

    # Also ignore archive artifacts (built via "mix archive.build").
    *.ez

    # Ignore package tarball (built via "mix hex.build").
    <appname>-*.tar

    # If NPM crashes, it generates a log, let's ignore it too.
    npm-debug.log

    # The directory NPM downloads your dependencies sources to.
    /assets/node_modules/

    # Since we are building assets from assets/,
    # we ignore priv/static. You may want to comment
    # this depending on your deployment strategy.
    /priv/static/

# Set up README.md in root

# Set up CHANGELOG.md in root

    # Changelog
    All notable changes to this project will be documented in this file.

    The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
    and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

    ## [Unreleased]

# Set up RELEASE.md in root

    # Release Instructions

      1. Bump version in related files below
      2. Run `mix my.release`, which does the following:
          * checks format
          * runs tests & checks coverage
          * ensures prod env is up-to-date
          * preps phoenix app for release
          * creates the release
      3. Commit, tag and push code

    ## Files with version

      * `CHANGELOG`
      * `mix.exs`


# Run Locally
		cd <appname>/<appname_web> 
		mix phx.server 
		iex -S mix phx.server


# Git Flow
		git add -A 
		git commit -m "message" 
		git log
