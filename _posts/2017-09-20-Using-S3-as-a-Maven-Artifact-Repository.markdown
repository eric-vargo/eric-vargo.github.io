---
layout: post
title:  "Using S3 as a Maven Artifact Repository"
date:   2019-02-11 16:02:54
---
These are my notes on setting up S3 as an artifact repository. Why do this?  Well, because everyone hates Nexus.  It's clunky and it just smells like an overly heavyweight solution to a fairly straightforward problem.  S3 exists so why not use what is readily available instead of running a local Nexus repository?

Configuring gradle to use the “maven-publish” plugin was pretty easy.  It involved creating a named publication to the project’s “publishing.publications” container (“myProjectNameHere” in the example below).  Here is a sample configuration that publishes the boot jar artifact to an S3 repository:

    repositories {
        mavenLocal()
        mavenCentral()
        maven {
            name "S3"
            url "s3://{S3-bucket-name}"
            authentication {
                awsIm(AwsImAuthentication)
            }
        }
    }

    publishing {
        publications {
            myProjectNameHere(MavenPublication) {
                from components.java
            }
        }
        repositories {
            maven {
                url "s3://{S3-bucket-name}/{S3-folder-name}”
                credentials(AwsCredentials) {
    	            accessKey "$s3RepoAccessKey"
    	            secretKey "$s3RepoSecretKey"
    	        }
            }
        }
    }

I decided to use gradle variables for the S3 access keys, allowing the current user (or CI server) to provide it’s own authentication credentials using gradle.properties.  This avoids having to hard-code credentials in the build file.
_(edit: AWS authorization has been simplified since I first set this up. You can now use environment variables for the AWS access keys)_

The name chosen for the publication name is used as a partial string of the generated tasks the maven-publish plugin creates.
generatePomFileForMyProjectNameHerePublication and publishMyProjectNameHerePublicationToMavenRepository
This shows in logging output so choosing a name appropriate for the project and artifact being create is useful when debugging builds.

By default, the plugin will populate the identity values in the generated POM from the following gradle project values:

    group = project.group
    artifactId = defaults to project.name (in settings.gradle) (can be overridden by the 
    version = project.version

Overriding the default identity values is trivial: specify the groupId, artifactId or version attributes when configuring.  This is achieved by added those properties to the MavenPublication object as such:

    publications {
        myProjectNameHere(MavenPublication) {
            groupId 'org.gradle.sample'
            artifactId 'project1-sample'
            version '1.1'

            from components.java
        }  
    }


Gradle Commands
---------------
These commands are not specific to the type of repository, S3 or otherwise.  This is the standard command given by the maven-publish plugin.

To fully build and publish a project: 

    gradle clean build publish

The publish task does not fully build a project as of this writing (Gradle version 2.12), so the build task must be explicitly run.  You can fix this issue by adding these lines to the buildfile:

    publish.dependsOn build
    publishToMavenLocal.dependsOn build


To publish to local maven repository when developing (“installing” the module in maven parlance): 

    gradle publishToMavenLocal

Getting artifacts from the S3 Repository
----------------------------------------

I had to setup the S3 bucket to serve static web pages in order to be able to use it as a maven repository.  You can then use a bucket policy to restrict access in any number of ways.  For starters just to get the ball rolling, I restricted access to our office IP address:

    {
    	"Version": "2012-10-17",
    	"Id": "some-id-here",
    	"Statement": [
    		{
    			"Sid": "IPAllow",
    			"Effect": "Allow",
    			"Principal": "*",
    			"Action": "s3:*",
    			"Resource": "arn:aws:s3:::myProjectNameHere-artifacts/*",
    			"Condition": {
    				"IpAddress": {
    					"aws:SourceIp": “999.999.999.999/32"
    				}
    			}
    		}
    	]
    }

Relevant documentation in Gradle:
https://docs.gradle.org/current/userguide/publishing_maven.html
https://docs.gradle.org/current/userguide/dependency_management.html#sec:supported_transport_protocols

