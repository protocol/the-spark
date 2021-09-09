- Instructions https://medium.com/crowdbotics/a-complete-one-by-one-guide-to-install-docker-on-your-mac-os-using-homebrew-e818eb4cfc3
- Install [[virtualbox]] (needed by docker to emulate hosts)
	- `brew install --cask docker virtualbox`
	-
	  > When you do fail, turn on System Preference and see if ‘System software from developer “Oracle America, inc” was blocked from loading.’ If you see that message, click Allow button and try to install again.
- Install [[docker-machine]] which is like docker backend as far as I can tell
	- `brew install docker-machine`
- Create a docker machine
	- `docker-machine create --driver virtualbox default`
- Not sure why but internet seems to suggest we need this
	-
	  ```
	  docker-machine restart
	  docker-machine env default
	  eval $(docker-machine env default)
	  ```
- Install docker client
	- `brew install docker`
- Hello world
	- `docker run hello-world`