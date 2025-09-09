# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **Hiram Village Community Calendar** - a Ruby on Rails application designed to consolidate and display community events for the village of Hiram. The platform aggregates events from multiple sources (RSS feeds, web scraping, social media) and provides a unified calendar interface with user submission capabilities.

## Architecture & Technology Stack

### Current Setup
- **Ruby**: 2.6.10 (planned upgrade to Ruby 3.3.8)
- **Rails**: 6.1.3.2 (planned upgrade to Rails 8.0.2.1)
- **Database**: PostgreSQL with PostGIS for geographic data
- **Background Jobs**: Sidekiq with Redis
- **Frontend**: Planned Hotwire (Turbo + Stimulus) with Tailwind CSS
- **Testing**: RSpec, FactoryBot, Capybara, VCR (90%+ coverage target)
- **Version Control**: chruby for Ruby version management

### Key Dependencies (Planned)
```ruby
# Authentication & Authorization
gem 'devise'
gem 'pundit'

# API & Integrations  
gem 'httparty'
gem 'feedjira'      # RSS parsing
gem 'nokogiri'      # Web scraping
gem 'koala'         # Facebook API
gem 'geocoder'      # Location services

# Frontend
gem 'turbo-rails'
gem 'stimulus-rails'
gem 'tailwindcss-rails'
gem 'view_component'
```

## Development Commands

### Ruby/Rails Management
```bash
# Switch Ruby versions (chruby)
chruby ruby-3.3.8

# Generate Rails application
rails new . --database=postgresql --css=tailwind --skip-git --force

# Database operations
rails db:create
rails db:migrate
rails db:seed
```

### Testing Commands
```bash
# Run full test suite
bundle exec rspec

# Run specific test file
bundle exec rspec spec/models/event_spec.rb

# Run tests with coverage
COVERAGE=true bundle exec rspec

# Run system tests
bundle exec rspec spec/system/
```

### Code Quality
```bash
# Run RuboCop linter
bundle exec rubocop

# Security scan
bundle exec brakeman

# Performance profiling
bundle exec rack-mini-profiler
```

### Background Jobs
```bash
# Start Sidekiq
bundle exec sidekiq

# Sidekiq web interface
bundle exec sidekiq-web
```

## Code Architecture

### Modular Design Principles
The application follows a modular architecture with domain-specific organization:

#### Core Domain Modules
- **Events Module** (`app/models/events/`): Main event models, categories, locations, organizers
- **Content Sources Module** (`app/models/sources/`): RSS feeds, scrapers, social media sources
- **Moderation Module** (`app/models/moderation/`): User submissions, approval workflows

#### Service Objects Pattern
- **Event Services** (`app/services/events/`): Event creation, updates, duplicate detection, searching
- **Integration Services** (`app/services/integrations/`): RSS importers, web scrapers, social media APIs
- **Moderation Services** (`app/services/moderation/`): Content screening, spam detection, approval workflows

#### ViewComponent Architecture
- **Calendar Components** (`app/components/calendar/`): Week/month/day views, event cards
- **Event Components** (`app/components/events/`): Detail views, lists, forms
- **Shared Components** (`app/components/shared/`): Filters, pagination, notifications

### Key Integration Points

#### Data Sources
- **RSS Feeds**: Automated pulling from partner organizations (Hiram College, Village Government, Monroe's Orchards)
- **Web Scraping**: Regular scraping of local websites using Nokogiri
- **Social Media**: Facebook Graph API integration for public events
- **User Submissions**: Public submission form with moderation workflow

#### Background Processing
- **Daily Jobs**: RSS feed updates, web scraping, content moderation
- **Real-time**: Social media monitoring every 2 hours
- **Deduplication**: Intelligent event matching to prevent duplicates

## Test-Driven Development (TDD) Approach

### Testing Strategy
The project follows strict TDD with a test pyramid structure:
- **Unit Tests (30%)**: Models, services, helpers
- **Integration Tests (30%)**: Service interactions, API endpoints  
- **Request Tests (30%)**: Controller specs, API responses
- **System Tests (10%)**: Full user workflows, critical paths

### VCR for External APIs
Use VCR cassettes for testing external API integrations:
```ruby
# spec/services/integrations/rss_importer_spec.rb
context "with VCR cassettes" do
  it "imports from real feed URLs", :vcr
end
```

### Factory Patterns
```ruby
# spec/factories/events.rb
FactoryBot.define do
  factory :event do
    title { Faker::Lorem.sentence }
    start_date { 1.week.from_now }
    association :category
    association :location
    
    trait :recurring do
      recurring { true }
      recurrence_rule { "FREQ=WEEKLY;BYDAY=MO" }
    end
  end
end
```

## Development Workflow

### Git Strategy
```
main
├── develop
│   ├── feature/event-system
│   ├── feature/submission-portal
│   ├── feature/rss-integration
│   └── feature/scraping-engine
└── hotfix/critical-bug
```

### Pull Request Requirements
- All tests passing (90%+ coverage for new code)
- RuboCop compliance
- Brakeman security scan passing
- Performance impact assessment

## Implementation Status

### Current Phase: Foundation & Setup
The project is currently in the initial setup phase. Key immediate tasks:
1. Ruby 3.3.8 + Rails 8 upgrade
2. Database schema design and migrations
3. RSpec test framework configuration
4. Core event model implementation

### Planned Development Phases
1. **Phase 1**: Core Event System (Events CRUD, Calendar Views)
2. **Phase 2**: User Submission System (Public forms, Moderation)  
3. **Phase 3**: Data Integration Layer (RSS, Scraping, Social Media)
4. **Phase 4**: Advanced Features (Search, User Accounts)
5. **Phase 5**: Performance & Polish (Optimization, SEO)
6. **Phase 6**: Beta Testing & Launch

## Key Business Requirements

### Primary Features
- **Event Aggregation**: Automatically collect events from multiple sources
- **Calendar Views**: Week/month/list views with intuitive navigation  
- **Public Submissions**: Community members can submit events for moderation
- **Content Moderation**: All submissions reviewed before publication
- **Geographic Focus**: Events centered around Hiram village area

### Performance Targets
- Page load times < 2 seconds
- 99.5% uptime requirement
- Mobile-first responsive design
- 500+ weekly users within 6 months

### Integration Partners
- Hiram Christian Church
- Hiram College  
- Monroe's Orchards
- Village Government
- Local Facebook groups/Nextdoor

## Important Notes

### Ruby/Rails Upgrade Path
The project is currently on Ruby 2.6.10 / Rails 6.1.3.2 but is architected for easy upgrade to Ruby 3.3.8 / Rails 8.0.2.1. Prioritize the upgrade for better performance and modern Rails features.

### Security Considerations
- Never commit API keys or sensitive credentials
- Use environment variables for all external service configurations
- Implement proper input validation and sanitization for user submissions
- Regular security scanning with Brakeman

### Monitoring & Performance
- Application Performance Monitoring (Scout/New Relic planned)
- Error tracking (Sentry integration planned)  
- Background job monitoring via Sidekiq web interface
- Database query optimization to prevent N+1 queries