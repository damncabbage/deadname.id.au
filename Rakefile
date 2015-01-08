desc 'Create a new draft post; rake post TITLE="A New Post"'
task :post do
  now   = Time.now

  title = ENV['TITLE'] or raise "Missing TITLE=... argument!"
  slug  = "#{now.strftime("%F")}-#{title.downcase.gsub(/[^\w]+/, '-')}"
  file  = File.join(File.dirname(__FILE__), '_posts', slug + '.markdown')

  File.open(file, "w") do |f|
    f << <<-EOS.gsub(/^    /, '')
    ---
    layout: post
    title: "#{title}"
    date: "#{now.strftime("%F %H:%M")}"
    published: false
    categories:
    - Example
    ---

    EOS
  end

  puts "Generated #{file} - opening..."
  exec (ENV['EDITOR'] || 'vim'), file
end
