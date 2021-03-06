# GitHub API and GraphQL Client #

This package provides a generic GitHub API client (`GithubApiClient`) as well as `Codable`-like GitHub GraphQL querying and decoding based on an object's properties.

These two targets automatically handle:
- [Authenticating as a GitHub App](https://docs.github.com/en/developers/apps/building-github-apps/authenticating-with-github-apps#authenticating-as-a-github-app)
- Building GraphQL queries from Swift objects
- Decoding Swift objects from GraphQL responses

The GraphQL interface is handled by defining your query using a simple Result Builder interface:
```swift
static let query = Node(type: "ProjectV2") {
	Field("title")
	Field("url")
}
```

Using this tree, the actual GraphQL query string can be automatically generated, as can the translation from the JSON result into an instance of your type.

GraphQL functionality is currently designed only for GitHub's GraphQL API and may not function correctly with any other GraphQL server.
Additionally, this is still in **extremely early development** and may not yet support even all GitHub GraphQL querying operations.

The GitHub API client has explicit helper methods for a limited subset of the full API, currently including:
- Basic app authentication
- GraphQL queries
- Posting issue comments and reactions

However, any arbitrary API call may be made, with authentication automatically injected.


## Quick Start ##

Pre-built examples:
- [GitHub Actions Webhook example](./Examples/GithubActionsWebhookClient): command-line utility that invokes an Actions webhook on a configured repository as a GitHub App (does not use any GraphQL functionality, only the GitHub API client)
- [GitHub Projects GraphQL example](./Examples/GithubProjectsGraphqlClient): command-line utility that fetches information about a GitHub Projects (V2) item given an input node ID, authenticating as a GitHub App


Add to your `Package.Swift`:
```swift
...
	dependencies: [
		...
		.package(url: "https://github.com/MPLew-is/github-graphql-client", branch: "main"),
	],
	targets: [
		...
		.target(
			...
			dependencies: [
				...
				.product(name: "GithubApiClient", package: "github-graphql-client"),
				.product(name: "GithubGraphqlQueryable", package: "github-graphql-client"),
			]
		),
		...
	]
]
```

Query a test object from the GitHub GraphQL API:
```swift
import GithubGraphqlClient
import GithubGraphqlQueryable

@main struct GithubGraphqlExample {
	struct ProjectV2 {
		static let query = Node(type: "ProjectV2") {
			Field("title")
		}

		@Value var title: String
	}

	static func main() async throws{
		let privateKey: String = """
			-----BEGIN RSA PRIVATE KEY-----
			...
			-----END RSA PRIVATE KEY-----
			""" // Replace with your GitHub App's private key
		let client: GithubApiClient = try await .init(
			appId: "123456", // Replace with your GitHub App ID
			privateKey: privateKey
		)

		let item = try await client.graphqlQuery(
			ProjectV2.self,
			id: "PVT_...", // Replace with the unique node ID for your GitHub Project (V2)
			for: "MPLew-is" // Replace with the user on which your app has been installed)
		)
	}
}
```
(See [the GraphQL client example](./Examples/GithubProjectsGraphqlClient) for more detailed instructions on how to set up a GitHub app and get the required authentication/configuration values)


## Targets provided ##

- `GithubGraphqlQueryable`: a protocol and associated types for automatic query generation and decoding from a GraphQL JSON response

- `GithubApiClient`: an abstraction layer around supported GitHub API endpoints (including the GraphQL one), automatically handling the authentication needed for a GitHub App
