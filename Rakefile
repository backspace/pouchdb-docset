require 'nokogiri'
require 'sqlite3'
require 'uri'

DOCUMENTATION_URL = 'http://pouchdb.com/'

task default: [
  :fetch_documentation,
  :strip_surroundings,
  :build_index,
  :add_api_table_of_contents,
  :copy_plist,
  :copy_readme,
  :create_archive,
  :generate_icons
]

task :fetch_documentation do
  target_directory = documents_path
  wget_command = <<-END_OF_COMMAND
    wget --mirror
         --convert-links
         --page-requisites
         -nH
         --exclude-directories=blog,2014
         --directory-prefix=#{target_directory}
         #{DOCUMENTATION_URL}
    END_OF_COMMAND
  system wget_command.gsub(/\n/, "")
end

task :strip_surroundings do
  Dir.glob(documents_path("/**/*.html")).each do |path|
    strip_surroundings(path)
  end
end

task :build_index do
  db_path = resources_path "/docSet.dsidx"
  db = SQLite3::Database.new db_path

  create_docset_table(db)
  parse_docset_into_db(db)
end

task :add_api_table_of_contents do
  path = documents_path("/api.html")
  file = File.open(path)
  document = Nokogiri::HTML(file)

  document.css("a.h2.anchor").each do |anchor|
    new_anchor = Nokogiri::XML::Node.new "a", document
    new_anchor['class'] = 'dashAnchor'
    new_anchor['name'] = "//apple_ref/cpp/Method/#{URI.escape(anchor.content).gsub("/", "%2F")}"

    anchor.add_previous_sibling(new_anchor)
  end

  file.close

  File.open(path, 'w') {|f| f.print(document.to_html)}
end

task :copy_plist do
  system "cp Info.plist #{contents_path}"
end

task :copy_readme do
  system "cp README.md dist/README.md"
end

task :create_archive do
  system "cd dist; tar --exclude='.DS_Store' -czf PouchDB.tgz PouchDB.docset"
end

task :generate_icons do
  favicon_path = documents_path("/static/img/mark.svg")

  {16 => "icon.png", 32 => "icon@2x.png"}.each do |size, filename|
    convert_command = <<-END_OF_COMMAND
      convert -resize x#{size}
              -gravity center
              -extent #{size}x#{size}
              #{favicon_path}
              dist/#{filename}
    END_OF_COMMAND
    system convert_command.gsub(/\n/, "")
  end
end

private
def strip_surroundings(path)
  file = File.open(path)
  document = Nokogiri::HTML(file)

  document.css("header, div.icons").remove

  file.close

  File.open(path, 'w') {|f| f.print(document.to_html)}
end

def create_docset_table(db)
  db.execute <<-SQL
    CREATE TABLE IF NOT EXISTS searchIndex(id INTEGER PRIMARY KEY, name TEXT, type TEXT, path TEXT);
    CREATE UNIQUE INDEX anchor ON searchIndex (name, type, path);
  SQL
end

def parse_docset_into_db(db)
  parse_file_into_db "/api.html", "Method", db
  parse_file_into_db "/guides/index.html", "Guide", db
  parse_file_into_db "/learn.html", "Guide", db
end

def parse_file_into_db(relative_file_path, type, db)
  file_path = documents_path(relative_file_path)
  file = File.open(file_path)
  document = Nokogiri::HTML(file)
  file.close

  parent_directory = "#{relative_file_path.split("/")[0..-2].join("/")}"

  document.css('#sidebar ul.nav li a').each do |item|
    name = item.content
    path = "#{parent_directory}/#{item.attr('href')}"[1..-1]
    insert_statement = <<-SQL
      INSERT OR IGNORE INTO searchIndex(name, type, path) VALUES ("#{name}", "#{type}", "#{path}");
    SQL

    db.execute insert_statement
  end
end

def contents_path(path = "")
  "#{Dir.getwd}/dist/PouchDB.docset/Contents"
end

def resources_path(path = "")
  "#{contents_path}/Resources#{path}"
end

def documents_path(path = "")
  "#{resources_path("/Documents")}#{path}"
end
