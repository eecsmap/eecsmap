Run in docker

(on my mac pro 2013 x64)
```
brew install docker colima
brew services start colima
brew services info colima
docker ps

docker run -it --rm haskell
```

Or use https://play.haskell.org/

Ref:
* https://hub.docker.com/_/haskell
* https://play.haskell.org/

To run a program in a file.

```
cat <<eof > test.hs
sayHello :: String -> IO()
sayHello x = putStrLn("hello, " ++ x ++ "!")
eof
docker run -it -v .:/usr/src --rm haskell
ghci> :load test.hs
ghci> sayHello "friend"
hello, friend!
```
