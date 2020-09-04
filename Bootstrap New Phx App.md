# Required Software

    Unix (needed for ASDF. E.G. WSL on Windows)
    ASDF (will install Nodejs, Erlang and Elixir with this)

# Initial Setup

    mix new <appname> --umbrella 
    cd <appname>/apps 
    mix new <appname> 
    mix phx.new.web <appname_web> 
    cd .. 
    git init 
    git commit -am "initial commit"

# Umbrella inter-dep

    Edit your web app mix.exs file to list your other app as a dependency:
      defp deps do
        [{:<appname>, in_umbrella: true}]
      end

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

# Set up Dialyzer / Dialyxir

    create `dialyzer.ignore-warnings` in umbrella root dir

    put any dialyzer warnings that you want to ignore inside this file, e.g.:

      :0: Unknown function 'Elixir.ExUnit.Callbacks':'__merge__'/3
      :0: Unknown function 'Elixir.ExUnit.CaseTemplate':'__proxy__'/2

# Set up ExCoverAlls

    for an umbrella project, add the following to the mix.exs `project` for each app
        test_coverage: [tool: ExCoveralls]


    create `coveralls.json` in umbrella root dir

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

# Set up Credo

    run `mix credo.gen.config` in umbrella root

# Set up Drab

    run `mix drab.install` in web application root

    or, if that doesn't work, e.g. if you have a non-standard installation, follow this:

    https://hexdocs.pm/drab/readme.html#manual-installation

# Add Redirect to router.ex 

    alias MyAppWeb.{PageController, Router}


    get "/", Router.Redirect, to: "/newpath"

    defmodule Redirect do
      def init(opts), do: opts

      def call(conn, opts) do
        conn
        |> Phoenix.Controller.redirect(opts)
        |> halt()
      end
    end


# Useful for controllers

    plug :defaults

    def index(conn, _params) do
      render(
        conn,
        "index.html",
        page_title: "My Page Title"
      )
    end

    defp defaults(conn, _params) do
      conn
      |> assign(:search_results, [])
      |> assign(:links, [])
      |> assign(:something_else, nil)
    end

# Update endpoint.ex gzip to be prod only

    plug Plug.Static,
      at: "/",
      from: :panarion,
      gzip: Mix.env() == :prod,
      only: ~w(css fonts images js favicon.ico robots.txt)

# Set up app for Releases (on-prem server)

    add a `releases` property to your umbrella mix.exs project definition:
    You only need to mention entry-point applications here.
       releases: [
          appname_release: [
            version: "0.1.0",
            applications: [app1: :permanent, app2: :permanent]
          ]
        ]


    Rename /config/prod.secret.exs (build-time, when release is assembled)
    To: /config/releases.exs (run-time)
    And update the contenets, here is an example, for deployment with gigalixir:

          import Config

          secret_key_base =
            System.get_env("SECRET_KEY_BASE") ||
              raise """
              environment variable SECRET_KEY_BASE is missing.
              You can generate one by calling: mix phx.gen.secret
              """

          config :panarion, PanarionWeb.Endpoint,
            # http: [port: String.to_integer(System.get_env("PORT") || "4000")],
            http: [port: {:system, "PORT"}],
            url: [host: System.get_env("APP_NAME") <> ".gigalixirapp.com", port: 443],
            secret_key_base: secret_key_base,
            server: true



    Update prod.exs
        change 
          url host from "example.com" to "localhost"

        add
          # Do not print debug messages in production
          config :logger, level: :info

        remove
          import_config "prod.secret.exs"

# Run Locally
		cd <appname>/<appname_web> 
		mix phx.server 
		iex -S mix phx.server


# Git Flow
    Show commits
      git log

    Show tags
      git tag
    
    Push changes
      git status
      git add -A 
      git commit -m "message" 
      git tag -a v1.4 -m "my version 1.4"
      git push

# Phoenix development workflow

## Controllers

    index - renders a list of all items of the given resource type
    show - renders an individual item by id
    new - renders a form for creating a new item
    create - receives params for one new item and saves it in a datastore
    edit - retrieves an individual item by id and displays it in a form for editing
    update - receives params for one edited item and saves it to a datastore
    delete - receives an id for an item to be deleted and deletes it from a datastore

