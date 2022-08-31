`<?xml version=``"1.0"` `encoding=``"UTF-8"``?>`

`<project name=``"api"` `default``=``"build"``>`

        `<target name=``"build"` `depends=``"make_runtime,phpcs-ci,phploc,pdepend,phpcb,phpunit,phpdox,phpcpd"``/>`

        `<property name=``"version-m"`  `value=``"1.1"` `/>`

        `<property name=``"version"`    `value=``"1.1.0"` `/>`

        `<property name=``"stability"`  `value=``"stable"` `/>`

        `<property name=``"releasenotes"` `value=``""` `/>`

        `<property name=``"tarfile"`     `value=``"${phing.project.name}.${buildnumber}.${buildid}.tar.gz"` `/>`

        `<property name=``"pkgfile"`     `value=``"${phing.project.name}.${version}.tgz"` `/>`

        `<property name=``"distfile"`    `value=``"dist/${tarfile}"` `/>`

        `<property name=``"tests.dir"` `value=``"test"` `/>`

        `<fileset id=``"api.tar.gz"` `dir=``"."``>`

            `<``include` `name=``"test/**"``/>`

            `<``include` `name=``"*.php"``/>`

            `<``include` `name=``"*.xml"``/>`

        `</fileset>`

        `<target name=``"make_runtime"``>`

                `<``mkdir` `dir=``"${project.basedir}/Runtime"` `/>`

                `<``mkdir` `dir=``"${project.basedir}/build/logs"` `/>`

                `<``mkdir` `dir=``"${project.basedir}/build/pdepend"` `/>`

                `<``mkdir` `dir=``"${project.basedir}/build/code-browser"` `/>`

        `</target>`

        `<target name=``"phpcs"` `description=``"Find coding standard violations using PHP_CodeSniffer"``>`

                `<``exec` `executable=``"phpcs"``>`

                        `<arg value=``"--standard=${project.basedir}/build/phpcs.xml"` `/>`

                        `<arg value=``"--ignore=autoload.php"` `/>`

                        `<arg path=``"${project.basedir}/"` `/>`

                `</``exec``>`

        `</target>`

        `<target name=``"phpcs-ci"` `description=``"Find coding standard violations using PHP_CodeSniffer"``>`

                `<``exec` `executable=``"phpcs"` `output=``"${project.basedir}/build/build.log"``>`

                        `<arg value=``"--report=checkstyle"` `/>`

                        `<arg value=``"--report-file=${project.basedir}/build/logs/checkstyle.xml"` `/>`

                        `<arg value=``"--standard=${project.basedir}/build/phpcs.xml"` `/>`

                        `<arg value=``"--ignore="` `/>`

                        `<arg path=``"${project.basedir}/"` `/>`

                `</``exec``>`

        `</target>`

        `<target name=``"phploc"` `description=``"Measure project size using PHPLOC"``>`

                `<``exec` `executable=``"phploc"``>`

                        `<arg value=``"--log-csv"` `/>`

                        `<arg value=``"${project.basedir}/build/logs/phploc.csv"``/>`

                        `<arg path=``"${project.basedir}/"``/>`

                `</``exec``>`

        `</target>`

        `<target name=``"pdepend"` `description=``"Calculate software metrics using PHP_Depend"``>`

                `<``exec` `executable=``"pdepend"``>`

                        `<arg value=``"--jdepend-xml=${project.basedir}/build/logs/jdepend.xml"``/>`

                        `<arg value=``"--jdepend-chart=${project.basedir}/build/pdepend/dependencies.svg"``/>`

                        `<arg value=``"--overview-pyramid=${project.basedir}/build/pdepend/overview-pyramid.svg"``/>`

                        `<arg path=``"${project.basedir}/"``/>`

                `</``exec``>`

        `</target>`

        `<target name=``"phpmd"` `description=``"Perform project mess detection using PHPMD"``>`

                `<``exec` `executable=``"phpmd"``>`

                        `<arg path=``"${project.basedir}/"``/>`

                        `<arg value=``"text"``/>`

                        `<arg value=``"${project.basedir}/build/phpmd.xml"``/>`

                `</``exec``>`

        `</target>`

        `<target name=``"phpmd-ci"` `description=``"Perform project mess detection using PHPMD"``>`

                `<``exec` `executable=``"phpmd"``>`

                        `<arg path=``"${project.basedir}/"``/>`

                        `<arg value=``"xml"``/>`

                        `<arg value=``"${project.basedir}/build/phpmd.xml"``/>`

                        `<arg value=``"--reportfile"``/>`

                        `<arg value=``"${project.basedir}/build/logs/pmd.xml"``/>`

                `</``exec``>`

        `</target>`

        `<target name=``"phpcpd"` `description=``"Find duplicate code using PHPCPD"``>`

                `<``exec` `executable=``"phpcpd"``>`

                        `<arg value=``"--log-pmd"``/>`

                        `<arg value=``"${project.basedir}/build/logs/pmd-cpd.xml"``/>`

                        `<arg path=``"${project.basedir}/"``/>`

                `</``exec``>`

        `</target>`

        `<target name=``"phpdox"` `description=``"Generate API documentation using phpDox"``>`

                `<``exec` `executable=``"phpdox"``/>`

        `</target>`

        `<target name=``"phpunit"` `description=``"Run unit tests with PHPUnit"``>`

                `<``exec` `executable=``"phpunit"` `/>`

        `</target>`

        `<target name=``"test"` `description=``"Run PHPUnit tests"``>`

            `<phpunit haltonerror=``"true"` `haltonfailure=``"true"` `printsummary=``"true"``>`

            `<batchtest>`

            `<fileset dir=``"${tests.dir}"``>`

                `<``include` `name=``"**/*Test.php"` `/>`

            `</fileset>`

            `</batchtest>`

            `</phpunit>`

        `</target>`

        `<target name=``"phpcb"` `description=``"Aggregate tool output with PHP_CodeBrowser"``>`

                `<``exec` `executable=``"phpcb"``>`

                        `<arg value=``"--log"``/>`

                        `<arg path=``"${project.basedir}/build/logs"``/>`

                        `<arg value=``"--source"``/>`

                        `<arg path=``"${project.basedir}/"``/>`

                        `<arg value=``"--output"``/>`

                        `<arg path=``"${project.basedir}/build/code-browser"``/>`

                `</``exec``>`

        `</target>`

        `<target name=``"check"` `description=``"Check variables"` `>`

            `<fail unless=``"version"` `message=``"Version not defined!"` `/>`

            `<fail unless=``"buildnumber"` `message=``"buildnumber not defined!"` `/>`

            `<fail unless=``"buildid"` `message=``"buildid not defined!"` `/>`

            `<``delete` `dir=``"dist"` `failonerror=``"false"` `/>`

            `<``mkdir` `dir=``"dist"` `/>`

        `</target>`

        `<target name=``"tar"` `depends=``"check"` `description=``"Create tar file for release"``>`

            `<``echo` `msg=``"Creating distribution tar for ${phing.project.name} ${version}"``/>`

            `<``delete` `file=``"${distfile}"` `failonerror=``"false"``/>`

            `<tar destfile=``"${distfile}"` `compression=``"gzip"``>`

                `<fileset refid=``"api.tar.gz"``/>`

            `</tar>`

        `</target>`

`</project>`