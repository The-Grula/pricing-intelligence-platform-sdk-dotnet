# Grula Pricing Intelligence Platform SDK for .NET

[![NuGet](https://img.shields.io/nuget/v/Grula.PricingIntelligencePlatform.Sdk.svg)](https://www.nuget.org/packages/Grula.PricingIntelligencePlatform.Sdk/)
[![Build Status](https://github.com/The-Grula/pricing-intelligence-platform-sdk-dotnet/workflows/Build%20&%20Publish/badge.svg)](https://github.com/The-Grula/pricing-intelligence-platform-sdk-dotnet/actions)

A .NET SDK for the Grula Pricing Intelligence Platform API, providing easy access to pricing policies, drivers, and price calculations.

## Features

- **Price Calculations**: Get prices based on price drivers and currency
- **Strongly Typed**: Generated from OpenAPI specification for type safety
- **Async/Await Support**: Full async support for all operations
- **Authentication**: Built-in support for Bearer token authentication

## Installation

Install the package via NuGet Package Manager:

```bash
dotnet add package Grula.PricingIntelligencePlatform.Sdk
```

Or via Package Manager Console:

```powershell
Install-Package Grula.PricingIntelligencePlatform.Sdk
```

## Quick Start

### Setup with Dependency Injection (Recommended)

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using System.Net.Http.Headers;
using Grula.PricingIntelligencePlatform.Sdk;

// In Program.cs or Startup.cs
var builder = Host.CreateApplicationBuilder(args);

builder.Services.AddHttpClient<GrulaApiClient>(client =>
{
    client.BaseAddress = new Uri("https://api.grula.net");
    client.DefaultRequestHeaders.Authorization = 
        new AuthenticationHeaderValue("Bearer", builder.Configuration["GrulaApi:ApiKey"]);
});

var host = builder.Build();
```

### Using the Client in a Service

```csharp
public class PricingService
{
    private readonly GrulaApiClient _grulaClient;

    public PricingService(GrulaApiClient grulaClient)
    {
        _grulaClient = grulaClient;
    }

    public async Task<decimal> GetProductPriceAsync(string product, string region, string currency = "USD")
    {
        var priceQuery = new GetPriceByPriceDriversQuery
        {
            EnvironmentId = Guid.Parse("your-environment-id"),
            CurrencyThreeLetterCode = currency,
            PricingDate = DateTime.UtcNow,
            PriceDrivers = new[]
            {
                new PriceDriver { Name = "Product", Value = product },
                new PriceDriver { Name = "Region", Value = region }
            }
        };

        var price = await _grulaClient.GetPriceByPriceDriversAsync(priceQuery);
        return (decimal)price.Amount.Amount;
    }

    public async Task<List<decimal>> GetMultiplePricesAsync(List<(string product, string region, string currency)> requests)
    {
        var pricesQuery = new GetPricesByPriceDriversQuery
        {
            EnvironmentId = Guid.Parse("your-environment-id"),
            PriceRequests = requests.Select(r => new PriceRequest
            {
                CurrencyThreeLetterCode = r.currency,
                PricingDate = DateTime.UtcNow,
                PriceDrivers = new[]
                {
                    new PriceDriver { Name = "Product", Value = r.product },
                    new PriceDriver { Name = "Region", Value = r.region }
                }
            }).ToArray()
        };

        var prices = await _grulaClient.GetPricesByPriceDriversAsync(pricesQuery);
        return prices.Select(p => (decimal)p.Amount.Amount).ToList();
    }
}
```

### Get Price by Drivers

```csharp
var priceQuery = new GetPriceByPriceDriversQuery
{
    EnvironmentId = Guid.Parse("your-environment-id"),
    CurrencyThreeLetterCode = "USD",
    PricingDate = DateTime.UtcNow,
    PriceDrivers = new[]
    {
        new PriceDriver { Name = "Product", Value = "Premium" },
        new PriceDriver { Name = "Region", Value = "US-East" }
    }
};

var price = await client.GetPriceByPriceDriversAsync(priceQuery);
Console.WriteLine($"Price: {price.Amount.Amount} {price.Amount.CurrencyThreeLetterCode}");
```

### Get Multiple Prices (Batch Request)

```csharp
var pricesQuery = new GetPricesByPriceDriversQuery
{
    EnvironmentId = Guid.Parse("your-environment-id"),
    PriceRequests = new[]
    {
        new PriceRequest
        {
            CurrencyThreeLetterCode = "USD",
            PricingDate = DateTime.UtcNow,
            PriceDrivers = new[]
            {
                new PriceDriver { Name = "Product", Value = "Basic" },
                new PriceDriver { Name = "Region", Value = "US-West" }
            }
        },
        new PriceRequest
        {
            CurrencyThreeLetterCode = "EUR",
            PricingDate = DateTime.UtcNow,
            PriceDrivers = new[]
            {
                new PriceDriver { Name = "Product", Value = "Premium" },
                new PriceDriver { Name = "Region", Value = "EU-Central" }
            }
        }
    }
};

var prices = await client.GetPricesByPriceDriversAsync(pricesQuery);
foreach (var price in prices)
{
    Console.WriteLine($"Price: {price.Amount.Amount} {price.Amount.CurrencyThreeLetterCode}");
}
```

### Basic Setup (Alternative)

```csharp
using System.Net.Http.Headers;
using Grula.PricingIntelligencePlatform.Sdk;

var httpClient = new HttpClient
{
    BaseAddress = new Uri("https://api.grula.net")
};

httpClient.DefaultRequestHeaders.Authorization = 
    new AuthenticationHeaderValue("Bearer", "your-api-key-here");

var client = new GrulaApiClient(httpClient);
```

## Configuration

### HttpClient Configuration

You can configure the HttpClient with additional settings:

```csharp
var httpClient = new HttpClient
{
    BaseAddress = new Uri("https://api.grula.net"),
    Timeout = TimeSpan.FromSeconds(30)
};

httpClient.DefaultRequestHeaders.Authorization = 
    new AuthenticationHeaderValue("Bearer", "your-api-key-here");

var client = new GrulaApiClient(httpClient);
```

### Dependency Injection

Register the client in your DI container:

```csharp
services.AddHttpClient<GrulaApiClient>(client =>
{
    client.BaseAddress = new Uri("https://api.grula.net");
    client.DefaultRequestHeaders.Authorization = 
        new AuthenticationHeaderValue("Bearer", configuration["GrulaApi:ApiKey"]);
});
```

### Logging

The SDK supports logging through the HttpClient:

```csharp
services.AddHttpClient<GrulaApiClient>()
    .AddLogger();
```

## Error Handling

The SDK throws exceptions for HTTP errors. Handle them appropriately:

```csharp
try
{
    var price = await client.GetPriceByPriceDriversAsync(priceQuery);
}
catch (ApiException ex)
{
    Console.WriteLine($"API Error: {ex.StatusCode} - {ex.Message}");
}
catch (HttpRequestException ex)
{
    Console.WriteLine($"Network Error: {ex.Message}");
}
```

## API Reference

### Price Calculations
- `GetPriceByPriceDriversAsync()` - Get a single price
- `GetPricesByPriceDriversAsync()` - Get multiple prices (max 100 requests)

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

For support and questions:

- Create an issue on [GitHub](https://github.com/The-Grula/pricing-intelligence-platform-sdk-dotnet/issues)
- Visit the [API documentation](https://api.grula.net)

## Changelog

### v1.0.0
- Initial release
- Full API coverage for Grula Pricing Intelligence Platform
- Support for all pricing operations
- Strongly typed models generated from OpenAPI specification
