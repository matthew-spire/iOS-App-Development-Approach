# Approach to iOS App Development

## Setup

- Each step informs the next
- Think "Big Picture" about the goal (purpose) of the app
  - What, if any, information are we trying to display
  - What, if any, information are we trying to collect
  - Auth requirements?
  - Etc.
- Think of what kind of architecture and-or design patterns to use
  - Goal is to architect the app in such a way that the app is scalable, extensible, maintainable, and easy to test &rarr; Pick the right architecture/design pattern for the job
  - MVC
    - Only for small and simple apps
    - No separation of concerns and you are mixing business logic with UI/View logic &rarr; Not easy to scale, extend, maintain, and or test
  - MVVM (w/ Repository)
    - My preference because it makes the app scalable, extensible, etc.
    - Dependency Injection (DI)
    - Separation of concerns
  - Etc. (MVP, MVI, ...)
- Project Structure
  - Start setting up the project structure based off the architecture and-or design pattern chosen
  - After creating the root/base level project using Xcode, I would switch to the terminal/command line to create the project's directories and files &rarr; Make sure this is possible
  - `ProjectName` &rarr; The root directory that Xcode created for you
    - Contains the main `ProjectNameApp.swift` file, which is the entry point for your app
  - `ProjectName/Views` &rarr; The directory that will contain all the SwiftUI views for the app
    - `FirstView.swift` &rarr; The first view
    - `SecondView.swift` &rarr; The second view
    - Etc.
  - `ProjectName/ViewModels` &rarr; This directory will contain the ViewModels for the app
    - `FirstViewModel.swift` &rarr; The ViewModel for the first view
    - Etc.
  - `ProjectName/Models` &rarr; This directory will contain data models for the app
    - E.g., `Character.swift` &rarr; The model representing a character (name should relate to what you are working with)
  - `ProjectName/Services` &rarr; This directory will contain the services and repository classes for the app
    - `ProjectAPI.swift` &rarr; Protocol for performing data operations (fetching data)
    - `ProjectAPIImplementation.swift` &rarr; Class implementing the `ProjectAPI` protocol for making API calls
    - `ProjectRepository.swift` &rarr; Class for handling the interaction between the ViewModels and the `ProjectAPI`
  - `ProjectName/Utils` &rarr; This directory will contain any utility classes, extensions, and constants for the app
    - `Constants.swift` &rarr; A file for storing constant values, like API URLs
    - `Extensions.swift` &rarr; A file for storing helpful Swift extensions
  - `ProjectName/Assets.xcassets` &rarr; This is the asset catalog that Xcode created for you. You can add images and icons here, such as the app icon and placeholder image for missing images
  - `ProjectNameTests` &rarr; This directory contains the test target for your app. You can write unit tests for your ViewModel(s), services, and repositories here
- Build the App
  - Models &rarr; Services &rarr; ViewModels and Views
  - FOcus on one thing, or screen, at a time

## Model

- 30,000-foot view: 
  - The model is a representation of the data, including its properties
  - The model is responsible for decoding JSON data and storing it in a structured format that can be used throughout the app
- Interaction w/ other parts of the app and separation of concerns:
  - The model is part of the data representation layer in the app
  - The model is responsible for decoding and storing the fetched JSON data, and is independent of the views and the networking layer &rarr; This separation of concerns allows for cleaner and more maintainable code, as each layer has a specific purpose and can be updated independently
- Creating the model:
  - Look at the JSON &rarr; Use a JSON beautifier if necessary
  - Is the information you need an array of objects associated with an object?
    - E.g., `"RelatedTopics":[{"key1":"value1", "key2":"value2", etc.}, {...}, etc.]` &rarr; The item on the left, which will be its own model, is made up of the items in the array on the right, which will be its own model
  - Code:
    ```
    import Foundation

    struct ModelName: Identifiable, Decodable {
        // Create what we will reference in our code
        // let firstItem = String

        // Create a relationship between what is in the JSON and what we will use in our code
        enum CodingKeys: String, CodingKey {
            // case firstItem = "FirstItem"
        }

        // Create a custom initializer that, effectively, maps the actual key (property) name that is within the JSON to what we are going to use in our app
        init(from decoder: Decoder) throws {
            // Create a container to hold the key-value pairs from the JSON data
            let container = try decoder.container(keyedBy: CodingKeys.self)
            // Decode the values for the properties
            // firstItem = try container.decode(String.self, forKey: .firstItem)
        }
    }
    ```

## ProjectAPI (Services)

- 30,000-foot view:
  - A protocol that defines a contract for performing data operations
  - Provides an abstraction that allows different implementations to perform data operations while conforming to the same interface
- Interaction w/ other parts of the app and separation of concerns:
  - Used by the Repository, which is responsible for handling the interaction between the ViewModel and the data operations service
  - The separation of concerns allows the ViewModel to make data requests without being aware of the specific implementation of the `ProjectAPI` &rarr; Ensures the ViewModel is only concerned with coordinating the data between the model and the view, while the Repository handles data operations and any necessary processing
- Creating the `ProjectAPI`:
  - What are we trying to do in terms of data operations? What HTTP request(s) will we be making? &rarr; Determine the functions that we are defining
  - The data that is being received or sent, what form is it taking? &rarr; Needs to take the form of something defined in our Models
  - Code:
    ```
    import Foundation
    import Combine

    protocol ProjectAPI {
        // Define a function for each API endpoint we plant to hit
        // func nameOfFunction(parameter: ParameterType) -> AnyPublisher<ReturnType, Error>
    }
    ```

## ProjectAPIImplementation (Services)

- 30,000-foot view:
  - A class that conforms to the `ProjectAPI` protocol, implementing the functions to perform data operations to and from the API
- Interaction w/ other parts of the app and separation of concerns:
  - The `ProjectAPIImplementation` is used by the `ProjectRepository`, which interacts with the ViewModel(s) &rarr; This separation of concerns allows the ViewModel to make data requests without knowing the details of the API implementation
  - The `ProjectAPIImplementation` is responsible for data operations with the API, while the `ProjectRepository` handles any interaction with the ViewModel(s) and any additional processing that may be required
- Creating the `ProjectAPIImplementation`:
  - Needs to implement the functions of the `ProjectAPI` (this time with the actual function body)
  - Includes the base URL
    - Should be moved to Utils/Constants.swift?
  - Code:
    ```
    import Foundation
    import Combine

    class ProjectAPIImplementation: ProjectAPI {
        // Define the base URL as a private constant

       // Implement the functions of the `ProjectAPI`
       // Create a URL by appending the parameter to the base URL - Use a guard let statement to safely unrap the optional URL
       // Use URLSessions on the request/URL to perform the desired data operation(s)
    }
    ```

## ProjectRepository (Services)

- 30,000-foot view:
  - The `ProjectRepository` is a class responsible for managing the interaction between the ViewModel and the `ProjectAPI`
  - The `ProjectRepository` performs data operations using the `ProjectAPI` instance
- Interaction w/ other parts of the app and separation of concerns:
  - The `ProjectRepository` is used by the ViewModel(s) to perform data operations
  - The ViewModel interacts with the Repository, which in turn interacts with the `ProjectAPI`
  - This separation of concerns allows the ViewModel to (request) perform data operations without knowing the details of the API implementation &rarr; Helps to keep the ViewModel focused on the presentation logic
- Creating the `ProjectRepository`:
  - Code:
    ```
    import Foundation
    import Combine

    class ProjectRepository {
        // A reference to an object that conforms to the ProjectAPI protocol
        private let projectAPI: ProjectAPI

        // Initializes the ProjectRepository with an object conforming to the ProjectAPI protocol
        init (projectAPI: ProjectAPI) {
            self.projectAPI = projectAPI
        }

        // Do the following for each function in ProjectAPI
        // Function implementation
        func nameOfFunction(parameter: ParameterType) -> AnyPublisher<ReturnType, Error> {
            projectAPI.nameOfFunction(parameter: parameter)
        }
    }
    ```

## ViewModel

- Creating the ViewModel:
  - Code:
    ```
    import Foundation
    import Combine

    // Define the ViewModel class, which conforms to the ObservableObject protocol. This allows SwiftUI views to observe changes in this ViewModel
    class ViewModel: ObservableObject {
        // A published property that holds an array of our models, allowing the View to observe and react to changes
        // @Published private(set) var items: [Items] = []

        // A reference to the ProjectRepository, which is responsible for data operations with the API
        private let projectRepository: ProjectRepository

        // A set of any cancellable objects used to store and manage Combine subscriptions
        private var cancellables = Set<AnyCancellable>()

        // The initializer that takes an optional ProjectRepository instance and initializes the ViewModel with it.
        // Also calls the function for initial data operations
        init(projectRepository: ProjectRepository = ProjectRepository(projectAPI: ProjectAPIImplementation())) {
            self.projectRepository = projectRepository
            nameOfFunction(parameter: parameterValue)
        }

        // Implement the function that you defined in the initializer
        private func nameOfFunction(parameter: ParameterType) {
            // Call the method on the ProjectRepository with the provided parameter
            projectRepository.nameOfFunction(parameter: parameter)
                // Ensure that any updates to the published properties are performed on the main thread
                .receive(on: DispatchQueue.main)
                // Create a sink subscriber with a completion and a value handler
                .sink(
                    // Handle the completion event (may be able to ignore in certain cases)
                    receiveCompletion: { _ in },
                    // Handle the value event, which updates the properties of the model with the new data
                    receiveValue: { items in self.items = items }
                )
                // Store the subscription in the cancellables set to manage its lifecycle
                .store(in: &cancellables)
        }
    }
    ```

## View

- Code:
  ```
  import SwiftUI

  // Define the View struct, which conforms to the View protocol
  struct NameOfView: View {
    // Create a @StateObject to manage the ViewModel instance
    @StateObject private var viewModel = ViewModel()

    // Define the body property, which is a computed property that returns a view hierarchy
    var body: some View {
        // Create a VStack that will contain what you want to display
        VStack {
            // Get the data from the ViewModel
        }
    }
  }
  ```