# F1 Laps API Documentation Guide

## Overview

The F1 Laps API provides access to Formula 1 lap time data and analytics. This guide covers the API documentation using RSwag, which generates OpenAPI (Swagger) specifications from RSpec tests.

## Setup

### 1. Dependencies

Add to your Gemfile:
```ruby
group :development, :test do
  gem 'rswag-specs'
end

gem 'rswag-api'
gem 'rswag-ui'
```

Run:
```bash
bundle install
```

### 2. Installation

Run the following generators in order:
```bash
rails generate rswag:api:install
rails generate rswag:ui:install
rails generate rswag:specs:install
```

This creates:
- `config/initializers/rswag_api.rb`
- `config/initializers/rswag_ui.rb`
- `spec/swagger_helper.rb`
- Mounts RSwag in `config/routes.rb`

### 3. Configuration

The API is configured in three main files:

1. **swagger_helper.rb**:
```ruby
RSpec.configure do |config|
  config.openapi_root = Rails.root.join('swagger').to_s
  config.openapi_specs = {
    'v1/swagger.yaml' => {
      openapi: '3.0.1',
      info: {
        title: 'F1 Laps API',
        version: 'v1',
        description: 'API for accessing Formula 1 lap time data'
      },
      paths: {},
      servers: [
        {
          url: 'http://{defaultHost}',
          variables: {
            defaultHost: {
              default: 'localhost:3000'
            }
          }
        }
      ]
    }
  }
  config.openapi_format = :yaml
end
```

2. **rswag_ui.rb**:
```ruby
Rswag::Ui.configure do |c|
  c.openapi_endpoint '/api-docs/v1/swagger.yaml', 'API V1 Docs'
end
```

3. **rswag_api.rb**:
```ruby
Rswag::Api.configure do |c|
  c.openapi_root = Rails.root.join('swagger').to_s
end
```

## API Endpoints

### Health Check
- `GET /api/v1/health`
  - Returns API health status
  - Response: `{ status: "healthy", version: "v1", timestamp: "..." }`

### Lap Times
- `GET /api/v1/lap_times`
  - List all lap times
  - Query Parameters:
    - `driver_id` (optional): Filter by driver
    - `circuit_id` (optional): Filter by circuit
    - `lap_min` (optional): Minimum lap number
    - `lap_max` (optional): Maximum lap number

- `POST /api/v1/lap_times`
  - Create a new lap time
  - Required fields: `driver_id`, `circuit_id`, `time_ms`, `lap_number`

- `GET /api/v1/drivers/{driver_id}/lap_times`
  - Get lap times for a specific driver

- `GET /api/v1/circuits/{circuit_id}/lap_times`
  - Get lap times for a specific circuit

### Drivers
- `GET /api/v1/drivers`
  - List all drivers

- `POST /api/v1/drivers`
  - Create a new driver
  - Required fields: `name`, `code`

### Circuits
- `GET /api/v1/circuits`
  - List all circuits

- `POST /api/v1/circuits`
  - Create a new circuit
  - Required fields: `name`, `location`

## Testing & Documentation Generation

1. Write tests in `spec/requests/api/v1/` following RSwag conventions
2. Generate documentation:
   ```bash
   RAILS_ENV=test rake rswag:specs:swaggerize
   ```
3. Documentation is generated to `swagger/v1/swagger.yaml`

## Viewing Documentation

1. Start the Rails server:
   ```bash
   rails server
   ```

2. Visit http://localhost:3000/api-docs

The Swagger UI provides:
- Interactive API documentation
- Request/response schemas
- Test endpoints directly in the browser
- Example requests and responses

## Response Formats

### Success Responses
```json
// Single object
{
  "id": 1,
  "name": "Example",
  ...
}

// Collection
[
  {
    "id": 1,
    "name": "Example 1",
    ...
  }
]
```

### Error Responses
```json
{
  "errors": [
    {
      "code": "validation_error",
      "message": "Validation failed",
      "details": {
        "field_name": ["error message"]
      }
    }
  ]
}
```

## Best Practices

1. **Documentation First**: Write RSwag specs before implementing endpoints
2. **Complete Examples**: Include realistic example values in specs
3. **Error Cases**: Document error responses and edge cases
4. **Consistent Naming**: Use consistent parameter names across endpoints
5. **Version Control**: Keep documentation in version control with code
6. **Regular Updates**: Regenerate docs after API changes

## Troubleshooting

1. **Missing Documentation**
   - Ensure specs include proper RSwag DSL
   - Check file paths match actual endpoints
   - Verify `swagger_helper.rb` configuration

2. **UI Not Loading**
   - Check RSwag is mounted in `routes.rb`
   - Verify server is running
   - Clear browser cache

3. **Test Failures**
   - Compare implementation with documentation
   - Check required parameters
   - Verify response formats

4. **Generation Issues**
   - Run in test environment
   - Check file permissions
   - Verify YAML syntax

## References

- [RSwag GitHub Repository](https://github.com/rswag/rswag)
- [OpenAPI Specification](https://swagger.io/specification/)
- [Swagger UI Configuration](https://swagger.io/docs/open-source-tools/swagger-ui/usage/configuration/)

# Implementing RSwag in a Rails API

This guide provides a detailed walkthrough for implementing RSwag to document an existing Rails API. We'll use a Formula 1 lap times API as our concrete example.

## Table of Contents

1. [Introduction](#introduction)
2. [Step 1: Add RSwag Gems](#step-1-add-rswag-gems)
3. [Step 2: Install RSwag Components](#step-2-install-rswag-components)
4. [Step 3: Configure RSwag](#step-3-configure-rswag)
5. [Step 4: Write Documentation Specs](#step-4-write-documentation-specs)
6. [Step 5: Generate Documentation](#step-5-generate-documentation)
7. [Step 6: View Documentation](#step-6-view-documentation)
8. [Common Scenarios](#common-scenarios)
9. [Troubleshooting](#troubleshooting)
10. [References](#references)

## Introduction

RSwag is a Ruby gem that adds Swagger (OpenAPI) documentation capabilities to Rails-based APIs. It consists of three components:

- **rswag-specs**: For defining and testing OpenAPI specifications
- **rswag-api**: For serving the OpenAPI JSON/YAML files
- **rswag-ui**: For providing the Swagger UI documentation interface

This approach allows you to:
- Write documentation as part of your test process
- Test that your API conforms to its documentation
- Generate up-to-date documentation automatically
- Provide an interactive interface for API users

## Step 1: Add RSwag Gems

Add the following gems to your Gemfile:

```ruby
group :development, :test do
  gem 'rswag-specs'
end

gem 'rswag-api'
gem 'rswag-ui'
```

Then install the gems:

```bash
bundle install
```

## Step 2: Install RSwag Components

Unlike some older documentation might suggest, RSwag's installation is split into separate generators. Run each generator separately:

```bash
rails generate rswag:api:install
rails generate rswag:ui:install
rails generate rswag:specs:install
```

This creates:
- `config/initializers/rswag_api.rb` - API configuration
- `config/initializers/rswag_ui.rb` - UI configuration 
- `spec/swagger_helper.rb` - Documentation testing configuration
- Mount points in `config/routes.rb` for both the API and UI components

## Step 3: Configure RSwag

### 3.1 Configure swagger_helper.rb

Edit `spec/swagger_helper.rb` to match your API:

```ruby
# Example from F1 Laps API
RSpec.configure do |config|
  config.openapi_root = Rails.root.join('swagger').to_s

  config.openapi_specs = {
    'v1/swagger.yaml' => {
      openapi: '3.0.1',
      info: {
        title: 'F1 Laps API',
        version: 'v1',
        description: 'API for accessing Formula 1 lap time data and analytics',
        contact: {
          name: 'API Support',
          email: 'support@f1laps.com'
        },
        license: {
          name: 'MIT',
          url: 'https://opensource.org/licenses/MIT'
        }
      },
      paths: {},
      components: {
        securitySchemes: {
          bearer_auth: {
            type: :http,
            scheme: :bearer,
            bearerFormat: 'JWT'
          }
        }
      },
      servers: [
        {
          url: 'http://{defaultHost}',
          variables: {
            defaultHost: {
              default: 'localhost:3000'
            }
          }
        }
      ]
    }
  }

  config.openapi_format = :yaml
end
```

Key configuration points:
- Set `openapi_root` to where you want the files generated
- Configure the title, version, and description of your API
- Add contact information for API support
- Set up security schemes if your API requires authentication
- Configure server information (development, staging, production)

### 3.2 Configure UI (rswag_ui.rb)

Edit `config/initializers/rswag_ui.rb` to customize the documentation UI:

```ruby
Rswag::Ui.configure do |c|
  c.openapi_endpoint '/api-docs/v1/swagger.yaml', 'API V1 Docs'
  
  # UI configuration options
  c.config_object['defaultModelsExpandDepth'] = 1
  c.config_object['displayRequestDuration'] = true
  c.config_object['docExpansion'] = 'list'
  c.config_object['filter'] = true
  c.config_object['tryItOutEnabled'] = true
end
```

> Note: The method `swagger_endpoint` is deprecated and replaced with `openapi_endpoint` in newer versions.

### 3.3 Configure API (rswag_api.rb)

The default configuration in `config/initializers/rswag_api.rb` is usually sufficient:

```ruby
Rswag::Api.configure do |c|
  c.openapi_root = Rails.root.join('swagger').to_s
end
```

This tells rswag-api where to find the OpenAPI files for serving.

## Step 4: Write Documentation Specs

Create spec files in the `spec/requests/api/v1/` directory (adjust path according to your API structure):

### 4.1 Health Endpoint Documentation

Let's start with a simple health check endpoint:

```ruby
# spec/requests/api/v1/health_spec.rb
require 'swagger_helper'

RSpec.describe 'Health API', type: :request do
  path '/api/v1/health' do
    get 'Get API health status' do
      tags 'Health'
      produces 'application/json'
      
      response '200', 'health status' do
        schema type: :object,
          properties: {
            status: { type: :string, enum: ['healthy'] },
            version: { type: :string },
            timestamp: { type: :string, format: 'date-time' }
          },
          required: ['status', 'version', 'timestamp']
        
        run_test!
      end
    end
  end
end
```

### 4.2 Resource Endpoint Documentation

For more complex endpoints, like our lap times resource:

```ruby
# spec/requests/api/v1/lap_times_spec.rb
require 'swagger_helper'

RSpec.describe 'Lap Times API', type: :request do
  path '/api/v1/lap_times' do
    get 'List all lap times' do
      tags 'Lap Times'
      produces 'application/json'
      
      parameter name: :driver_id, in: :query, type: :integer, required: false, description: 'Filter by driver ID'
      parameter name: :circuit_id, in: :query, type: :integer, required: false, description: 'Filter by circuit ID'
      parameter name: :lap_min, in: :query, type: :integer, required: false, description: 'Minimum lap number'
      parameter name: :lap_max, in: :query, type: :integer, required: false, description: 'Maximum lap number'
      
      response '200', 'lap times found' do
        schema type: :array,
          items: {
            type: :object,
            properties: {
              id: { type: :integer },
              driver_id: { type: :integer },
              circuit_id: { type: :integer },
              time_ms: { type: :integer },
              lap_number: { type: :integer },
              created_at: { type: :string, format: 'date-time' },
              updated_at: { type: :string, format: 'date-time' }
            },
            required: ['id', 'driver_id', 'circuit_id', 'time_ms', 'lap_number']
          }
        
        run_test!
      end
    end

    post 'Create a lap time' do
      tags 'Lap Times'
      consumes 'application/json'
      produces 'application/json'
      
      parameter name: :lap_time, in: :body, schema: {
        type: :object,
        properties: {
          driver_id: { type: :integer },
          circuit_id: { type: :integer },
          time_ms: { type: :integer },
          lap_number: { type: :integer }
        },
        required: ['driver_id', 'circuit_id', 'time_ms', 'lap_number']
      }
      
      response '201', 'lap time created' do
        let(:lap_time) { { driver_id: 1, circuit_id: 1, time_ms: 80000, lap_number: 1 } }
        run_test!
      end

      response '422', 'invalid request' do
        let(:lap_time) { { driver_id: 1 } }
        run_test!
      end
    end
  end

  # Document nested routes
  path '/api/v1/drivers/{driver_id}/lap_times' do
    get 'Get lap times for a specific driver' do
      tags 'Lap Times'
      produces 'application/json'
      
      parameter name: :driver_id, in: :path, type: :integer, required: true
      
      response '200', 'lap times found' do
        let(:driver_id) { 1 }
        schema type: :array,
          items: {
            type: :object,
            properties: {
              id: { type: :integer },
              circuit_id: { type: :integer },
              time_ms: { type: :integer },
              lap_number: { type: :integer },
              created_at: { type: :string, format: 'date-time' },
              updated_at: { type: :string, format: 'date-time' }
            }
          }
        run_test!
      end
    end
  end
end
```

For drivers and circuits, create similar spec files that match your controller implementations.

### Key Components of RSwag Specs:

1. **Path Operation**: Define the path and HTTP method
   ```ruby
   path '/api/v1/lap_times' do
     get 'Description' do
       # ...
     end
   end
   ```

2. **Tags**: Group related operations
   ```ruby
   tags 'Lap Times'
   ```

3. **Parameters**: Define query, path, and body parameters
   ```ruby
   parameter name: :driver_id, in: :query, type: :integer, required: false, description: 'Filter by driver'
   ```

4. **Request Body**: Define the request payload for POST/PUT/PATCH
   ```ruby
   parameter name: :lap_time, in: :body, schema: {
     type: :object,
     properties: { ... },
     required: ['driver_id', 'circuit_id', 'time_ms', 'lap_number']
   }
   ```

5. **Responses**: Define possible response codes and schemas
   ```ruby
   response '200', 'success' do
     schema type: :array, items: { ... }
     run_test!
   end
   ```

6. **Example Values**: Provide examples with the `let` syntax
   ```ruby
   let(:lap_time) { { driver_id: 1, circuit_id: 1, time_ms: 80000, lap_number: 1 } }
   ```

## Step 5: Generate Documentation

After writing your documentation specs, generate the OpenAPI specification file:

```bash
RAILS_ENV=test rake rswag:specs:swaggerize
```

This will:
1. Run your RSwag specs to validate the documentation
2. Generate the OpenAPI specification file at the configured location (e.g., `swagger/v1/swagger.yaml`)

You should run this command:
- After making changes to your API specifications
- As part of your CI/CD pipeline
- Before deploying to ensure documentation is up-to-date

## Step 6: View Documentation

Start your Rails server:

```bash
rails server
```

Then visit the Swagger UI at:
```
http://localhost:3000/api-docs
```

You'll see an interactive documentation interface where you can:
- Browse all API endpoints organized by tags
- See request parameters and response schemas
- Try out API endpoints directly from the browser
- View sample requests and responses

## Common Scenarios

### Authentication

If your API requires authentication, you can configure security schemes in `swagger_helper.rb`:

```ruby
components: {
  securitySchemes: {
    bearer_auth: {
      type: :http,
      scheme: :bearer,
      bearerFormat: 'JWT'
    }
  }
}
```

Then, add security requirements to operations:

```ruby
get 'Protected endpoint' do
  security [bearer_auth: []]
  # ...
end
```

### File Uploads

For file upload endpoints:

```ruby
post 'Upload file' do
  consumes 'multipart/form-data'
  
  parameter name: :file, in: :formData, type: :file, required: true
  
  response '200', 'file uploaded' do
    # ...
  end
end
```

### Reusing Schemas

For DRY schemas, define schema components in `swagger_helper.rb`:

```ruby
components: {
  schemas: {
    lap_time: {
      type: :object,
      properties: { ... },
      required: [...]
    }
  }
}
```

Then reference in your specs:

```ruby
schema '$ref' => '#/components/schemas/lap_time'
```

## Troubleshooting

### Missing Documentation

**Issue**: Endpoints don't appear in Swagger UI.  
**Solution**: 
- Ensure specs include proper RSwag DSL
- Verify file paths match your actual controller routes
- Check that your spec files are in `spec/requests/` (or configured path)

### UI Not Loading

**Issue**: Swagger UI page doesn't load or is empty.  
**Solution**: 
- Verify RSwag is mounted in `routes.rb`
- Check if your OpenAPI file was generated at the expected location
- Inspect browser console for JavaScript errors

### Test Failures

**Issue**: Rswag spec tests fail when running `rake rswag:specs:swaggerize`.  
**Solution**: 
- Compare your controller implementation with the documentation
- Check required parameters in both spec and controller
- Verify response format matches the documented schema
- Look for error details in the test output

### Generation Issues

**Issue**: OpenAPI file isn't generated correctly.  
**Solution**: 
- Run in test environment (`RAILS_ENV=test`)
- Check file permissions in the destination directory
- Verify your `swagger_helper.rb` configuration

## References

- [RSwag GitHub Repository](https://github.com/rswag/rswag)
- [OpenAPI Specification](https://swagger.io/specification/)
- [Swagger UI Configuration](https://swagger.io/docs/open-source-tools/swagger-ui/usage/configuration/)
- [RSpec Documentation](https://relishapp.com/rspec)
