pipeline {
	agent {
		node {
			label 'maven'
		}
	}
	environment {
		RHT_OCP4_DEV_USER = "hrjmgc"
		DEPLOYMENT_STAGE = 'shopping-cart-stage'
		DEPLOYMENT_PRODUCTION = 'shopping-cart-production'
	}
	stages {
		stage('Tests') {
			steps {
				sh './mvnw clean test'
			}
		}
		stage('Package') {
			steps {
				sh '''
					./mvnw package -DskipTests \
					-Dquarkus.package.type=uber-jar
				'''
				archiveArtifacts 'target/*.jar'
			}
		}
		stage('Build Image') {
			environment { QUAY = credentials('QUAY_USER') }
			steps {
				sh '''
					./mvnw quarkus:add-extension \
					-Dextensions="kubernetes,container-image-jib"
				'''
				sh '''
				./mvnw package -DskipTests \
				-Dquarkus.jib.base-jvm-image=quay.io/redhattraining/do400-java-alpine-openjdk11-jre:latest \
				-Dquarkus.container-image.build=true \
				-Dquarkus.container-image.registry=quay.io \
				-Dquarkus.container-image.group=$QUAY_USR \
				-Dquarkus.container-image.name=do400-deploying-environments \
				-Dquarkus.container-image.username=$QUAY_USR \
				-Dquarkus.container-image.password="$QUAY_PSW" \
				-Dquarkus.container-image.tag=build-${BUILD_NUMBER} \
				-Dquarkus.container-image.push=true
				'''
			}
		}
		stage ('Deploy - Stage') {
			environment {
				APP_NAMESPACE = "${RHT_OCP4_DEV_USER}-shopping-cart-stage"
				QUAY = credentials('QUAY_USER')
			}
			steps {
				sh """
					oc set image \
					deployment ${DEPLOYMENT_STAGE} \
					shopping-cart-stage=quay.io/${QUAY_USR}/do400-deploying-environments:build-${BUILD_NUMBER} \
					-n ${APP_NAMESPACE} --record
				"""
			}
		}
		stage('Deploy - Production') {
			environment {
				APP_NAMESPACE = "${RHT_OCP4_DEV_USER}-shopping-cart-production"
				QUAY = credentials('QUAY_USER')
			}
			input { message 'Deploy to production?' }
			steps {
				sh """
					oc set image \
					deployment ${DEPLOYMENT_PRODUCTION} \
					shopping-cart-production=quay.io/${QUAY_USR}/do400-deployingenvironments:
					build-${BUILD_NUMBER} \
					-n ${APP_NAMESPACE} --record	
				"""
			}
		}	
	}
}
