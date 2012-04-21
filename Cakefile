fs = require 'fs'
docs = "#{__dirname}/docs"

{basename, join} = require 'path'
{exec, spawn} = require 'child_process'
{series, parallel} = require 'async'
inspect = require('eyes').inspector
  stream: null
  pretty: false
  styles:
    all: 'magenta'


# Utility functions

pleaseWait = ->
  console.log "\nThis may take a while...\n"

handleError = (err) ->
  if err
    console.log "\nUnexpected error!\nHave you done `cake install`?\n"
    console.log err.stack

# execute some command quietly (without stdout)
sh = (command) -> (k) -> exec command, k

# Modified from https://gist.github.com/920698
runCommand = (name, args) ->
    proc = spawn name, args
    proc.stderr.on 'data', (buffer) -> console.log buffer.toString()
    proc.stdout.on 'data', (buffer) -> console.log buffer.toString()
    proc.on 'exit', (status) -> process.exit(1) if status != 0

# shorthand to runCommand with
command = (c, cb) ->
  runCommand "sh", ["-c", c]
  cb


# Install / reminder of prerequisites (for development).
# First-time setup.  Pygments is installed through pycco,
# or through other projects that use docco as well.
task 'install', "Run once: npm, bundler, pycco, etc.", ->
  pleaseWait()
  command "
    npm install
     && gem install bundler
     && bundle install
     && easy_install pycco
    "


# Check if any node_modules or gems have become outdated.
task 'outdated', "is all up-to-date?", ->
  pleaseWait()
  parallel [
    command "npm outdated"
    command "gem outdated"
  ], (err) -> throw err if err


# Usually follows `cake outdated`.
task 'update', "latest node modules & ruby gems - the lazy way", ->
  pleaseWait()
  parallel [
    command "npm update"
    command "bundle update"
  ], (err) -> throw err if err


# It's the local police at the project's root.
# Catches outdated modules that `cake outdated` doesn't report (major versions).
task 'police', "checks npm package & dependencies with `police -l .`", ->
  command "police -l ."


# Build manuals / gh-pages almost exactly like https://github.com/josh/nack does
task 'man', "Build unix man pages", ->
  fs.readdir "doc/", (err, files) ->
    for file in files when /\.md/.test file
      source = join "doc", file
      target = join "man", basename source, ".md"
      exec "ronn --pipe --roff #{source} > #{target}", (err) ->
        throw err if err


task 'pages', "Build pages / documents as well", ->

  buildMan = (callback) ->
    series [
      (sh "cake man")
      (sh "cp README.md doc/index.md")
      (sh 'echo "# UNLICENSE\n## LICENSE\n\n" > doc/UNLICENSE.md' )
      (sh "cat UNLICENSE >> doc/UNLICENSE.md")
      (sh "ronn -stoc -5 doc/*.md")
      (sh "mv doc/*.html pages/")
      (sh "rm doc/index.md")
      (sh "rm doc/UNLICENSE.md")
    ], callback

  buildAnnotations = (callback) ->
    series [
      (sh "rm -rf docs")
      (sh "docco bin/*.coffee")
      (sh "pycco -d docs/python bin/*.py")
      (sh "cp -r docs pages/annotations")
    ], callback

  build = (callback) ->
    parallel [buildMan, buildAnnotations], callback

  series [
    (sh "if [ ! -d pages ] ; then mkdir pages ; fi") # mkdir pages only if it doesn't exist
    (sh "rm -rf pages/*")
    build
  ], (err) -> throw err if err


task 'pages:publish', "Build pages and publish to gh-pages", ->

  checkoutBranch = (callback) ->
    series [
      (sh "rm -rf pages/")
      (sh "git clone -q -b gh-pages git@github.com:astrolet/precious.git pages")
      (sh "rm -rf pages/*")
    ], callback

  publish = (callback) ->
    series [
      (sh "cd pages/ && git add . && git commit -m 'rebuild manual' || true")
      (sh "cd pages/ && git push git@github.com:astrolet/precious.git gh-pages")
      (sh "rm -rf pages/")
    ], callback

  series [
    checkoutBranch
    (sh "cake pages") # NOTE: (invoke "pages") # doesn't work here after checkoutBranch
    publish
  ], (err) -> throw err if err
