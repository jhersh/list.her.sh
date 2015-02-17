task :setup do
	sh "bundle check --path=vendor/bundle || bundle install --jobs=4 --retry=2 --path=vendor/bundle"
end

task :serve do
	sh "bundle exec jekyll serve --watch --config _config.yml"
end

task :publish do
	system %{
		
		SITE="s3://list.her.sh/"
		
		bundle exec jekyll build --config _config.yml
		bundle exec htmlproof --check-favicon --check-html --verbose _site
		
		find _site/ -iname '*.html' -exec gzip -n --best {} +
		find _site/ -iname '*.xml' -exec gzip -n --best {} +
		
		for f in `find _site/ -iname '*.gz'`; do
			mv $f ${f%.gz}
		done
		
		# Sync GZip'd HTML and XML
		
		s3cmd sync --progress -M --acl-public \
		--add-header 'Content-Encoding:gzip' \
		_site/ $SITE \
		--exclude '*.*' \
		--include '*.html' --include '*.xml' \
		--verbose 
		
		# Sync all remaining files
		
		s3cmd sync --progress -M --acl-public \
		_site/ $SITE \
		--exclude '*.*' \
		--include '*.png' --include '*.css' --include '*.js' --include '*.txt' \
		--verbose 
		
	}
end