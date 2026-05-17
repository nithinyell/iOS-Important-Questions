Coordinator — the only one who pushes/presents screens. Owns UINavigationController.
ViewModel — holds state (@Published), calls the service, and tells the Coordinator when to navigate. Zero UIKit imports.
ViewController — renders UI and binds to ViewModel with Combine. Never pushes itself.
Model/Service — plain structs + a protocol so you can swap in a mock for tests.

The golden rule: when a user taps something → VC calls viewModel.didSelectItem() → ViewModel calls coordinator.showDetail() → Coordinator creates the next VC and pushes it. Navigation logic never leaks into VCs or ViewModels.

[Preview](https://htmlpreview.github.io/?https://github.com/nithinyell/iOS-Important-Questions/blob/main/MVVM-C.html)

Insta Like posts / Feed Mobile System Design
```
Data pagination solves 1000 records problem.
Image CDN + caching solves high-resolution image problem.
Virtualized list solves UI performance problem.
Prefetching solves smooth scrolling problem.
ViewModel state solves clean architecture problem.
```

```
import SwiftUI
import Foundation

// MARK: - Model

struct FeedResponse: Decodable {
    let feed: Feed
}

struct Feed: Decodable {
    let results: [FeedResult]
}

struct FeedResult: Decodable, Identifiable, Hashable {
    let id: String
    let name: String
    let artistName: String
    let artworkUrl100: String
}

// MARK: - Network

enum NetworkError: Error {
    case invalidURL
    case invalidResponse
    case decodingFailed
}

protocol NetworkServicing {
    func fetchData(from urlString: String) async throws -> Data
}

final class NetworkService: NetworkServicing {
    func fetchData(from urlString: String) async throws -> Data {
        guard let url = URL(string: urlString) else {
            throw NetworkError.invalidURL
        }

        let (data, response) = try await URLSession.shared.data(from: url)

        guard let httpResponse = response as? HTTPURLResponse,
              200..<300 ~= httpResponse.statusCode else {
            throw NetworkError.invalidResponse
        }

        return data
    }
}

// MARK: - Repository

protocol FeedRepositoryProtocol {
    func fetchFeed() async throws -> [FeedResult]
}

final class FeedRepository: FeedRepositoryProtocol {
    private let networkService: NetworkServicing

    private let urlString = "https://rss.marketingtools.apple.com/api/v2/us/music/most-played/50/albums.json"

    init(networkService: NetworkServicing) {
        self.networkService = networkService
    }

    func fetchFeed() async throws -> [FeedResult] {
        let data = try await networkService.fetchData(from: urlString)

        do {
            let response = try JSONDecoder().decode(FeedResponse.self, from: data)
            return response.feed.results
        } catch {
            throw NetworkError.decodingFailed
        }
    }
}

// MARK: - ViewModel

@MainActor
final class FeedViewModel: ObservableObject {
    @Published var posts: [FeedResult] = []
    @Published var isLoading = false
    @Published var errorMessage: String?

    private let repository: FeedRepositoryProtocol
    private let coordinator: FeedCoordinating

    init(
        repository: FeedRepositoryProtocol,
        coordinator: FeedCoordinating
    ) {
        self.repository = repository
        self.coordinator = coordinator
    }

    func fetchFeed() async {
        isLoading = true
        errorMessage = nil

        do {
            posts = try await repository.fetchFeed()
        } catch {
            errorMessage = "Something went wrong. Please try again."
        }

        isLoading = false
    }

    func didSelectPost(_ post: FeedResult) {
        coordinator.showDetails(post)
    }
}

// MARK: - Coordinator

enum FeedRoute: Hashable {
    case details(FeedResult)
}

protocol FeedCoordinating: AnyObject {
    func showDetails(_ post: FeedResult)
}

@MainActor
final class FeedCoordinator: ObservableObject, FeedCoordinating {
    @Published var path = NavigationPath()

    func start() -> some View {
        let networkService = NetworkService()
        let repository = FeedRepository(networkService: networkService)

        let viewModel = FeedViewModel(
            repository: repository,
            coordinator: self
        )

        return FeedView(viewModel: viewModel)
    }

    func showDetails(_ post: FeedResult) {
        path.append(FeedRoute.details(post))
    }

    @ViewBuilder
    func build(route: FeedRoute) -> some View {
        switch route {
        case .details(let post):
            FeedDetailView(post: post)
        }
    }
}

// MARK: - View

struct FeedView: View {
    @StateObject var viewModel: FeedViewModel

    var body: some View {
        Group {
            if viewModel.isLoading {
                ProgressView("Loading...")
            } else if let errorMessage = viewModel.errorMessage {
                Text(errorMessage)
                    .foregroundStyle(.red)
            } else {
                List(viewModel.posts) { post in
                    Button {
                        viewModel.didSelectPost(post)
                    } label: {
                        FeedRowView(post: post)
                    }
                    .buttonStyle(.plain)
                }
            }
        }
        .navigationTitle("Albums")
        .task {
            await viewModel.fetchFeed()
        }
    }
}

struct FeedRowView: View {
    let post: FeedResult

    var body: some View {
        HStack(spacing: 12) {
            AsyncImage(url: URL(string: post.artworkUrl100)) { image in
                image
                    .resizable()
                    .scaledToFill()
            } placeholder: {
                ProgressView()
            }
            .frame(width: 50, height: 50)
            .clipShape(RoundedRectangle(cornerRadius: 8))

            VStack(alignment: .leading, spacing: 4) {
                Text(post.name)
                    .font(.headline)

                Text(post.artistName)
                    .font(.subheadline)
                    .foregroundStyle(.secondary)
            }
        }
    }
}

// MARK: - Detail View

struct FeedDetailView: View {
    let post: FeedResult

    var body: some View {
        VStack(spacing: 16) {
            AsyncImage(url: URL(string: post.artworkUrl100)) { image in
                image
                    .resizable()
                    .scaledToFit()
            } placeholder: {
                ProgressView()
            }
            .frame(width: 160, height: 160)
            .clipShape(RoundedRectangle(cornerRadius: 16))

            Text(post.name)
                .font(.title2)
                .bold()

            Text(post.artistName)
                .font(.headline)
                .foregroundStyle(.secondary)

            Spacer()
        }
        .padding()
        .navigationTitle("Details")
    }
}

// MARK: - App

@main
struct PostsApp: App {
    @StateObject private var coordinator = FeedCoordinator()

    var body: some Scene {
        WindowGroup {
            NavigationStack(path: $coordinator.path) {
                coordinator.start()
                    .navigationDestination(for: FeedRoute.self) { route in
                        coordinator.build(route: route)
                    }
            }
        }
    }
}
```
