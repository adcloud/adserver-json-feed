require 'rubygems'
require "bundler/setup"
require "stringex"

# These tasks are taken from the Octopress project:
# http://octopress.org/
# https://github.com/imathis/octopress/blob/master/Rakefile

deploy_branch  = "gh-pages"
public_dir     = "output"    # compiled site directory
deploy_dir     = "_deploy"   # deploy directory (for Github pages deployment)
repo_url       = "git@github.com:adcloud/adserver-json-feed.git"

desc "copy dot files for deployment"
task :copydot, :source, :dest do |t, args|
  FileList["#{args.source}/**/.*"].exclude("**/.", "**/..", "**/.DS_Store", "**/._*").each do |file|
    cp_r file, file.gsub(/#{args.source}/, "#{args.dest}") unless File.directory?(file)
  end
end

desc "deploy public directory to github pages"
task :push do
  puts "## Deploying branch to Github Pages "
  (Dir["#{deploy_dir}/*"]).each { |f| rm_rf(f) }
  Rake::Task[:copydot].invoke(public_dir, deploy_dir)
  puts "\n## copying #{public_dir} to #{deploy_dir}"
  cp_r "#{public_dir}/.", deploy_dir
  cd "#{deploy_dir}" do
    system "git add ."
    system "git add -u"
    puts "\n## Commiting: Guides updated at #{Time.now.utc}"
    message = "Guides updated at #{Time.now.utc}"
    system "git commit -m \"#{message}\""
    puts "\n## Pushing generated #{deploy_dir} guides"
    system "git push origin #{deploy_branch} --force"
    puts "\n## Github Pages deploy complete"
  end
end

desc "Set up _deploy folder and deploy branch for Github Pages deployment"
task :setup, :repo do |t, args|
  rm_rf deploy_dir
  mkdir deploy_dir
  cd "#{deploy_dir}" do
    system "git init"
    system "echo 'Guides coming &hellip;' > index.html"
    system "git add ."
    system "git commit -m \"Initial commit\""
    system "git branch -m gh-pages"
    system "git remote add origin #{repo_url}"
  end
  puts "\n---\n## Now you can deploy with `rake deploy` ##"
end
