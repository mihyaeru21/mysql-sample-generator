
require 'faker'
require 'mysql2'

DB_NAME = ENV['LM_DBNAME'] || 'learning_mysql'
ROWS = (ENV['LM_ROWS'] || '500').to_i # 1000倍した行数が作られる
client = Mysql2::Client.new(
  host:     ENV['LM_HOST']     || '127.0.0.1',
  port:     ENV['LM_PORT']     || '3306',
  username: ENV['LM_USERNAME'] || 'root',
  password: ENV['LM_PASSWORD'] || 'hoge',
)

task default: :setup
task setup: [:init_database, :create_table, :insert_data] do
  puts 'done.'
end

task :init_database do
  client.query("DROP DATABASE IF EXISTS #{DB_NAME}")
  client.query("CREATE DATABASE #{DB_NAME}")
  puts "db is initialized."
end

task :use do
  client.query("use #{DB_NAME}")
end

task create_table: [:use] do
  def create_ddl(name)
    <<-"SQL"
      CREATE TABLE #{name} (
        id         INT UNSIGNED NOT NULL AUTO_INCREMENT,
        name       VARCHAR(32)  NOT NULL,
        birthday   DATE         NOT NULL,
        blood_type ENUM(
                      'A',
                      'B',
                      'O',
                      'AB'
                    )           NOT NULL,
        order_num  INT UNSIGNED NOT NULL,
        phone_num  VARCHAR(16)  NULL,
        PRIMARY KEY (id)
      ) ENGINE = InnoDB, CHARACTER SET = utf8mb4
    SQL
  end

  client.query(create_ddl('users'))
  client.query(create_ddl('slow_users'))
  client.query('ALTER TABLE users ADD INDEX idx_name (name)')
  client.query('ALTER TABLE users ADD INDEX idx_blood_type_birthday (blood_type, birthday)')
  client.query('ALTER TABLE users ADD INDEX idx_blood_type_name (blood_type, name)')
  client.query('ALTER TABLE users ADD INDEX idx_order_num_name (order_num, name)')
  client.query('ALTER TABLE users ADD INDEX idx_phone_num (phone_num)')
  puts "tables are created."
end

task insert_data: [:use] do
  Faker::Config.locale = :ja
  blood_types = %w(A B O AB)

  ROWS.times do |n|
    users = (1..1000).map do |i|
      name       = Faker::Name.name
      birthday   = Faker::Date.birthday.strftime('%F')
      blood_type = blood_types.sample
      order_num  = Faker::Number.between(1, 500_000_000)
      phone_num  = i.odd? ? 'NULL' : "'#{Faker::PhoneNumber.phone_number}'"
      "('#{name}','#{birthday}','#{blood_type}',#{order_num},#{phone_num})"
    end
    client.query("INSERT INTO users (name, birthday, blood_type, order_num, phone_num) values #{users.join(',')}")
    puts "#{(n + 1) * 1000} rows inserted."
  end

  client.query('insert into slow_users select * from users')
  puts 'users -> slow_users copied.'
end

