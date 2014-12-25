DOCUMENTATION_URL = 'http://pouchdb.com/'

task default: [
  :fetch_documentation
]

task :fetch_documentation do
  target_directory = "#{Dir.getwd}/PouchDB.docset/Contents/Resources/Documents"
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
