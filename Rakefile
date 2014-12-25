require 'nokogiri'
require 'sqlite3'

DOCUMENTATION_URL = 'http://pouchdb.com/'

task default: [
  :fetch_documentation,
  :strip_surroundings,
  :build_index,
]

task :fetch_documentation do
  target_directory = documents_path
  wget_command = <<-END_OF_COMMAND
    wget --mirror
         --convert-links
         --page-requisites
         -nH
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
  parse_guides_file_into_db "/guides/index.html", db
  parse_guides_file_into_db "/learn.html", db
end

def parse_guides_file_into_db(file_path, db)
  guides_document_path = documents_path(file_path)
  guides_file = File.open(guides_document_path)
  parsed_guides = Nokogiri::HTML(guides_file)
  guides_file.close

  parent_directory = "/#{file_path.split("/")[0..-2].join("/")}"
  puts parent_directory

  parsed_guides.css('#sidebar ul.nav li a').each do |guide|
    name = guide.content
    path = "#{parent_directory}/#{guide.attr('href')}"
    insert_statement = <<-SQL
      INSERT OR IGNORE INTO searchIndex(name, type, path) VALUES ('#{name}', 'Guide', '#{path}');
    SQL

    puts insert_statement
    db.execute insert_statement
  end
end

def resources_path(path = "")
  "#{Dir.getwd}/PouchDB.docset/Contents/Resources#{path}"
end

def documents_path(path = "")
  "#{resources_path("/Documents")}#{path}"
end
