# Initial Setup

mix new \<appname\> --umbrella \
cd \<appname\>/apps \
mix phx.new \<appname\> \
mix phx.new.web \<appname_web\> \
cd .. \
git init \
git commit -am "initial commit"



# Set Tool Versions with ASDF

https://alchemist.camp/episodes/asdf-language-versions \
cd \<appname\>

asdf current \
asdf plugin-add elixir -- add a new language \
asdf list -- list all installed languages \
asdf list-all elixir -- list all available  versions of a languages \
asdf install elixir \<version\> \
asdf global elixir \<version\>

asdf install nodejs \<version\> \
asdf install erlang \<version\> \
asdf install elixir \<version\>

bash \~/.asdf/plugins/nodejs/bin/import-release-team-keyring \
asdf local nodejs 14.9.0 \
export KERL_CONFIGURE_OPTIONS="--disable-debug --without-javac" \
asdf local erlang 23.0.2 \
asdf local elixir 1.10.4-otp-23 \
(elixir and erlang versions must match: 23 in this case)




# Package Setup

		(version numbers as of 2020-09-02. See https://hex.pm/packages/)
		 
		Umbrella: 
			  {:dialyxir, "\~\> 1.0", only: :test, runtime: false}, 
			  {:credo, "\~\> 1.4", only: [:dev, :test], runtime: false}, 
			  {:excoveralls, "\~\> 0.13.1", only: :test} 
		Web:
			  {:drab, "\~\> 0.10.5"},


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



# Package Flow




# Run Locally
		cd \<appname\>/\<appname_web\> 
		mix phx.server 
		iex -S mix phx.server


# Git Flow
		git add -A 
		git commit -m "message" 
		git log