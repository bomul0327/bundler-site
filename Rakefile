require "bundler/setup"

directory "vendor"
directory "vendor/bundler" => ["vendor"] do
  system "git clone git://github.com/bundler/bundler.git vendor/bundler"
  Dir.chdir("vendor/bundler") do
    sh "git remote add ruby-korea git@github.com:ruby-korea/bundler-site.git"
    sh "git fetch --all"
    sh "git checkout -b gh-pages ruby-korea/gh-pages"
  end
end

task :update_vendor => ["vendor/bundler"] do
  Dir.chdir("vendor/bundler") { sh "git fetch --all" }
end

desc "Pull in the man pages for the specified gem versions."
task :man => [:update_vendor] do
  %w(v1.0 v1.1 v1.2 v1.3 v1.5).each do |version|
    branch = (version[1..-1].split('.') + %w(stable)).join('-')

    mkdir_p "build/#{version}/man"

    Dir.chdir "vendor/bundler" do
      sh "git reset --hard HEAD"
      sh "git checkout origin/#{branch}"
      sh "ronn -5 man/*.ronn"
      cp(FileList["man/*.html"], "../../build/#{version}/man")
      sh "git clean -fd"
    end

  end

  # Make man pages for the latest version available at the top level, too.
  cp_r "build/v1.5/man", "build/man"
end

desc "Pulls in ISSUES.md from the master branch."
task :issues => [:update_vendor] do

  Dir.chdir "vendor/bundler" do
    sh "git reset --hard HEAD"
    sh "git checkout origin/master"
    cp "ISSUES.md", "../../source/issues.md"
  end

end

desc "Build the static site"
task :build => [:issues] do
  sh "middleman build --clean"
end

desc "Release the current commit to ruby-korea/bundler-site@gh-pages"
task :release => [:update_vendor, :build, :man, :issues] do
  commit = `git rev-parse HEAD`.chomp

  Dir.chdir "vendor/bundler" do
    sh "git reset --hard HEAD"
    sh "git checkout gh-pages"
    sh "git pull ruby-korea gh-pages"

    rm_rf FileList["*"]
    cp_r FileList["../../build/*"], "./"

    sh "git add -A ."
    sh "git commit -m 'ruby-korea/bundler-site@#{commit}'"
    sh "git push ruby-korea gh-pages"
  end
end

# Allow Heroku deploys to build the site (for previewing)
namespace :assets do
  task :precompile => [:build, :man]
end
