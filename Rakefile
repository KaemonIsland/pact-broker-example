require_relative "docker_configuration"
require "pact_broker/configuration"
require "pact_broker/initializers/database_connection"
require "pact_broker/tasks"

def create_database_configuration
  PactBroker.configuration.runtime_configuration = PactBroker.docker_configuration
  PactBroker.create_database_connection(PactBroker.configuration.database_configuration, PactBroker.configuration.logger)
end

def env(name)
  ENV[name] && ENV[name] != '' ? ENV[name] : nil
end

PactBroker::DB::MigrationTask.new do | task |
  task.database_connection = create_database_configuration
end

PactBroker::DB::VersionTask.new do | task |
  task.database_connection = create_database_configuration
end

PactBroker::DB::CleanTask.new do | task |
  task.database_connection = create_database_configuration
  task.logger = PactBroker.configuration.logger

  if env('PACT_BROKER_DATABASE_CLEAN_KEEP_VERSION_SELECTORS')
    require 'json'
    task.keep_version_selectors = JSON.parse(env('PACT_BROKER_DATABASE_CLEAN_KEEP_VERSION_SELECTORS'))
  end

  if env('PACT_BROKER_DATABASE_CLEAN_DELETION_LIMIT')
    task.version_deletion_limit = env('PACT_BROKER_DATABASE_CLEAN_DELETION_LIMIT').to_i
  end

  if env('PACT_BROKER_DATABASE_CLEAN_DRY_RUN')
    task.dry_run = env('PACT_BROKER_DATABASE_CLEAN_DRY_RUN') == 'true'
  end
end

PactBroker::DB::DeleteOverwrittenDataTask.new do | task |
  task.database_connection = create_database_configuration
  task.logger = PactBroker.configuration.logger

  if env('PACT_BROKER_DATABASE_CLEAN_OVERWRITTEN_DATA_MAX_AGE')
    task.max_age = env('PACT_BROKER_DATABASE_CLEAN_OVERWRITTEN_DATA_MAX_AGE').to_i
  end

  if env('PACT_BROKER_DATABASE_CLEAN_DELETION_LIMIT')
    task.deletion_limit = env('PACT_BROKER_DATABASE_CLEAN_DELETION_LIMIT').to_i
  end

  if env('PACT_BROKER_DATABASE_CLEAN_DRY_RUN')
    task.dry_run = env('PACT_BROKER_DATABASE_CLEAN_DRY_RUN') == 'true'
  end
end