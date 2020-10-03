# Required Software

    Unix (needed for ASDF. E.G. WSL on Windows)
    ASDF (will install Nodejs, Erlang and Elixir with this)

# Set up dev, build, app env:

    add these 5 lines to the end of your ~/.bashrc:
      export LC_ALL=en_US.UTF-8
      export LANG=en_US.UTF-8
      export LANGUAGE=en_US.UTF-8

      . $HOME/.asdf/asdf.sh
      . $HOME/.asdf/completions/asdf.bash

    install deps for asdf, elixir, erlang and nodejs (for build env)
    install asdf
    install elixir, erlang and nodejs with asdf
    bash ~/.asdf/plugins/nodejs/bin/import-release-team-keyring
    install posgresql if needed
    mix local.hex
    mix archive.install hex phx_new 1.4.11

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
    All Child Apps:
			  {:excoveralls, "~> 0.13.1", only: :test} 


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
        "cmd npm run deploy --prefix ./apps/<appname_web>/assets", 
        "phx.digest.clean", 
        "phx.digest" 
		  ],
		  "my.release": [ 
        "my.check", 
        "deps.get --only prod", 
        "cmd MIX_ENV=prod mix compile", 
        "my.phx",
        "cmd MIX_ENV=prod mix release" 
		  ],
      "my.run_release": [
        "cmd export SECRET_KEY_BASE=$(mix phx.gen.secret)",
        "cmd export PORT=4000",
        "cmd _build/prod/rel/my_app/bin/my_app start"
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


    create `coveralls.json` in each app root dir, not the umbrella root dir

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


    and add a controller redirection test:

        test "GET /", %{conn: conn} do
          conn = get(conn, "/")
          assert html_response(conn, 302) =~ "redirected"
        end

# Add core features to main layout: app.html.eex

    add custom stylesheets before your local, e.g. pure-css (https://purecss.io/start/), and also one for pure-css responsive grids:
    <link rel="stylesheet" href="https://unpkg.com/purecss@2.0.3/build/pure-min.css" integrity="sha384-cg6SkqEOCV1NbJoCu11+bm0NvBRc8IYLRGXkmNrqUBfTjmMYwNKPWBTIKyw9mHNJ" crossorigin="anonymous">
    <link rel="stylesheet" href="https://unpkg.com/purecss@2.0.3/build/grids-responsive-min.css">
    <link rel="stylesheet" href="<%= Routes.static_path(@conn, "/css/app.css") %>"/>

    add a Drab assign for page title: <title><%= @page_title %></title>

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

    add a `releases` property to your umbrella mix.exs project definition.
    You don't need to specify a version. It will use the version # of your umbrella project.
    You only need to mention entry-point applications here.
       releases: [
          release_name: [
            applications: [app1: :permanent, app2: :permanent],
            include_executables_for: [:unix],
            steps: [:assemble, :tar]
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


    SSL example: 
    Run `openssl req -new -x509 -nodes -out dev.crt -keyout dev.key` 
    inside the "priv/ssl" directory to generate dev key and cert.
    Or put your real ssl certs there, or use an environment variable here to set the path to them.

          config :gocu_web, GocuWeb.Endpoint,
            http: [port: {:system, "PORT"}],
            url: [host: "localhost", port: 4443],
            https: [
              port: 4443,
              cipher_suite: :strong,
              keyfile: "priv/ssl/dev.key",
              certfile: "priv/ssl/dev.crt"
            ],
            secret_key_base: secret_key_base,
            force_ssl: [hsts: true],
            server: true




    Update prod.exs
        change 
          url host from "example.com" to "localhost"

        add
          # Do not print debug messages in production
          config :logger, level: :info

        remove
          import_config "prod.secret.exs"

# Phoenix development workflow

## Controllers

    index - renders a list of all items of the given resource type
    show - renders an individual item by id
    new - renders a form for creating a new item
    create - receives params for one new item and saves it in a datastore
    edit - retrieves an individual item by id and displays it in a form for editing
    update - receives params for one edited item and saves it to a datastore
    delete - receives an id for an item to be deleted and deletes it from a datastore

# Static Assets

    <img src="<%= Routes.static_path(@conn, "/images/phoenix.png") %>" alt="Phoenix Framework Logo"/>

# Install SCSS / SASS

    1) Phoenix 1.4
    2) Add node-sass and sass-loader to /umbrella/apps/app_web/assets/package.json (devDependencies)
          "webpack": "4.4.0",
          "webpack-cli": "^3.3.2",
          "sass-loader": "7.3.1",
          "node-sass": "^4.9.0"
    3) cd /umbrella/apps/app_web/assets
    4) npm install node-sass sass-loader --save-dev
    5) rename assets/css/app.css to assets/css/app.scss
    6) add the sass-loader to the css file section in webpack.config.js and change the extension to .scss
            -        test: /\.css$/,
            -        use: [MiniCssExtractPlugin.loader, 'css-loader']
            +        test: /\.scss$/,
            +        use: [MiniCssExtractPlugin.loader, 'css-loader', 'sass-loader']

            plugins: [
              new MiniCssExtractPlugin({ filename: '../css/app.css' }), --- this needs to stay as .css!!!


    7) change import css from "../css/app.css" to import css from "../css/app.scss" in assets/js/app.js

    8) Template stylesheet needs to stay .css!!!

        <link rel="stylesheet" href="<%= Routes.static_path(@conn, "/css/app.css") %>"/>
          
              
          



# Create a Release

    Determine target triple, e.g.: x86_64-linux-gnu, by running gcc -dumpmachine

    Option 1: build on the same server that runs the app (2 machines: Dev, and Build/App)
    
        check out the code from git source control

        build a release

            mix my.release

        deploy it locally running under systemd.

            export SECRET_KEY_BASE=$(mix phx.gen.secret)
            export PORT=4000
            _build/prod/rel/my_app/bin/my_app start
  
        Like dev machine, the build server runs ASDF, using the versions of Erlang and Elixir in .tool-versions



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
      git push -u origin master                    initially to set remote tracking
      git push --tags                              if you have tags to push
      git push
   
    Amend Local
      git commit --amend -m "an updated commit message, adds new local changes to local commit." 

# Run Locally

    cd <appname>/<appname_web> 
    mix phx.server 
    iex -S mix phx.server

