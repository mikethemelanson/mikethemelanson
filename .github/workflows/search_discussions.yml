require 'octokit'
require 'json'

# Authenticate using the GITHUB_TOKEN environment variable
client = Octokit::Client.new(access_token: ENV['GITHUB_TOKEN'])

# Set organization and author from environment variables
org = ENV['ORG']
author = ENV['AUTHOR']

per_page = 100
page = 1
all_repos = []

puts "Fetching repositories for organization '#{org}'..."

# Fetch all repositories in the organization
loop do
  repos = client.org_repos(org, { type: 'all', per_page: per_page, page: page })
  break if repos.empty?
  all_repos.concat(repos)
  page += 1
end

puts "Found #{all_repos.size} repositories."

# Initialize an array to hold authored discussions
authored_discussions = []

all_repos.each do |repo|
  repo_name = repo.name
  owner = repo.owner.login

  puts "Searching discussions in repository '#{repo_name}'..."

  after = nil

  loop do
    query = <<-GRAPHQL
      query($owner: String!, $name: String!, $after: String) {
        repository(owner: $owner, name: $name) {
          discussions(first: 100, after: $after) {
            edges {
              node {
                title
                url
                createdAt
                author {
                  login
                }
              }
            }
            pageInfo {
              hasNextPage
              endCursor
            }
          }
        }
      }
    GRAPHQL

    variables = {
      owner: owner,
      name: repo_name,
      after: after
    }

    begin
      result = client.graphql(query, variables)
    rescue Octokit::Unauthorized
      puts "Authentication failed. Please check your GITHUB_TOKEN."
      exit 1
    rescue Octokit::Error => e
      puts "Error fetching discussions for repository '#{repo_name}': #{e.message}"
      break
    end

    discussions = result['repository']['discussions']

    discussions['edges'].each do |edge|
      node = edge['node']
      if node['author'] && node['author']['login'] == author
        authored_discussions << node.merge('repository' => repo_name)
      end
    end

    if discussions['pageInfo']['hasNextPage']
      after = discussions['pageInfo']['endCursor']
    else
      break
    end
  end
end

# Output the results
puts "\nAuthored Discussions:"
puts "-" * 40

authored_discussions.each do |discussion|
  puts "Repository: #{discussion['repository']}"
  puts "Title: #{discussion['title']}"
  puts "URL: #{discussion['url']}"
  puts "Created at: #{discussion['createdAt']}"
  puts "-" * 40
end

puts "\nTotal discussions found: #{authored_discussions.size}"
