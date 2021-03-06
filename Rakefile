task default: :pdf
book_regex = /(.*)-(\d*)-.*\.md/
book_regex2 = /(.*)-(\d*)-.*\.book/
pdfs = Dir.glob('**/*.md').map { |f| f.sub(/\.md$/, '.pdf') }
books = Dir.glob('**/*.md').map { |f| f if book_regex.match(f) }
books.compact!
pdfs -= books
csons = []

Dir.glob('data/**/*.cson') { |c| csons << c.sub(/\.cson$/, '.json') }
csons = csons.sort
task :cson => csons

rule '.json' => '.cson' do |csontojson|
  sh "cat #{csontojson.source} | gpp -H -x -DCSON=1 --include gpp.gppb >> build/tmp.cson"
  sh "cson2json build/tmp.cson >| #{csontojson.name}"
  rm 'build/tmp.cson'
end

pdfs = pdfs.sort
task pdf: pdfs
json2csvs = Dir.glob('data/forplotting/**/*.json').map { |f| f.sub /\.json$/, '.csv' }
task json2csv: json2csvs
unless Dir.glob('data/forplotting/*').empty?
  json2csvs.each do |f|
    cp f.sub(/\.csv$/, '.json'), f.sub(/\.csv$/, '.jsonfp')
  end
end
rule '.csv' => '.jsonfp' do |t|
  sh "json2csv convert #{t.source}"
  mv t.source.sub(/$/, '.csv'), t.name
  rm t.source
end

gpp_command = 'gpp -H -x -DTEX=1 --include gpp.gppb'

macros = Dir.glob('**/*.gpp').map { |f| f.sub(/\.gpp$/, '.gppb') }
task macro: macros

extensions = 'tex_math_single_backslash+footnotes+raw_tex+grid_tables+implicit_figures+fenced_code_blocks+definition_lists+pipe_tables+superscript+subscript+strikeout'
filters = ['pandoc-citeproc', 'pandoc-plantuml-filter'].join ' --filter '

rule '.gppb' => '.gpp' do |t|
  # Remove all whitespace and newlines from macro files
  sh "tr -d '\n' < #{t.source} > #{t.name}"
  sh "sed -r 's/>\s+</></g' #{t.name} > #{t.name + '.tmp'}"
  sh "mv #{t.name + '.tmp'} #{t.name}"
end

sh 'mkdir -vp build/books' unless Dir.exist? 'build/books'

books.each { |f| cp f, "build/books/#{File.basename(f, '.md') + '.book'}" unless /^build\//.match f }
booksp = Dir.glob('build/**/*.book').map { |f| f.sub /\.book$/, '.bookp' }
         .sort_by { |i| /(.*)-(\d*)-.*\.bookp/.match(i)[2].to_i  }
task bookp: booksp
rule '.bookp' => '.book' do |t|
  r = book_regex2.match t.source
  sh "cat #{t.source} >> #{r[1]}.md"
  pdfs << "#{r[1]}.pdf" # processed later on
end

rule '.pdf' => '.md' do |t|
  sh "cat #{t.source} | #{gpp_command} | pandoc  --template latex.tex -S -f markdown+#{extensions} -o #{t.name} --filter #{filters}" unless book_regex.match t.source
  mv t.name, '.' if /^build\//.match t.source
  rm t.source if /^build\//.match t.source
end

rule '.epub' => '.pdf' do |t|
  sh "cat build/books/#{File.basename(t.source, '.pdf') + '.md'} | #{gpp_command} | pandoc -S --template default.epub -f markdown+#{extensions} -o #{t.name} --filter #{filters} --epub-stylesheet epub.css" # unless book_regex.match t.source
end

rule '.pdf' => '.slide.md' do |t|
  sh "cat #{t.source} | #{gpp_command} | pandoc  -f markdown+#{extensions} -t beamer -o #{t.name} -S"
end
