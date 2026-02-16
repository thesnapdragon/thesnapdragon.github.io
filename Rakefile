namespace :blog do
  desc "Create a new draft"
  task :new, [:title] do |_, args|
    title = args[:title]
    date = Time.now.strftime("%Y-%m-%d")
    filename = "_drafts/#{title.downcase.gsub(/\s+/, "-")}.md"

    abort("#{filename} already exists!") if File.exist?(filename)

    File.open(filename, "w") do |file|
      file.puts "---"
      file.puts "layout: post"
      file.puts "title: #{title}"
      file.puts "date: #{date}T00:00:00Z"
      file.puts "categories: []"
      file.puts "---"
    end

    puts filename
  end

  desc "Release post"
  task :release, [:file_name] do |_, args|
    date = Time.now.strftime("%Y-%m-%d")

    File.rename("_drafts/#{args[:file_name]}", "_posts/#{date}-#{args[:file_name]}")
  end
end
