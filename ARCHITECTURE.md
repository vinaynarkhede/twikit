# Twikit Architecture Documentation

**Complete Multi-Layer Architecture Guide**

This document provides an in-depth analysis of the Twikit project architecture, explaining how the system works through multiple layers of abstraction: from the whole project structure down to individual functions.

---

## Table of Contents

1. [Overview & Philosophy](#1-overview--philosophy)
2. [Layer 1: Project-Level Architecture](#2-layer-1-project-level-architecture)
3. [Layer 2: File-Based Organization](#3-layer-2-file-based-organization)
4. [Layer 3: Class-Based Architecture](#4-layer-3-class-based-architecture)
5. [Layer 4: Function-Based Implementation](#5-layer-4-function-based-implementation)
6. [Data Flow Patterns](#6-data-flow-patterns)
7. [Authentication & Security](#7-authentication--security)
8. [API Interaction Patterns](#8-api-interaction-patterns)
9. [Design Patterns & Best Practices](#9-design-patterns--best-practices)
10. [Dependencies & External Integration](#10-dependencies--external-integration)

---

## 1. Overview & Philosophy

### What is Twikit?

Twikit is a **Python Twitter API scraper library** that enables programmatic interaction with Twitter/X **without requiring official API keys**. It achieves this through reverse-engineering Twitter's web interface and GraphQL APIs.

### Core Design Philosophy

1. **Asynchronous by Default**: Built on `asyncio` for high-performance concurrent operations
2. **User-Centric API**: Methods mirror user actions (login, tweet, follow) rather than raw API calls
3. **Client-Entity Pattern**: Central `Client` handles authentication/requests; entities (`Tweet`, `User`) represent domain objects
4. **Graceful Degradation**: Handles missing data, rate limits, and authentication challenges automatically
5. **No API Keys Required**: Uses web scraping techniques and reverse-engineered endpoints

### Key Characteristics

- **Version**: 2.3.3
- **Python**: 3.8+
- **Architecture Style**: Async Client-Server with Domain Models
- **Authentication**: Session-based (cookies) with automatic captcha solving
- **API Strategy**: GraphQL (primary) + REST API v1.1 (legacy/fallback)

---

## 2. Layer 1: Project-Level Architecture

### High-Level Component Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER APPLICATION                         │
│                    (async Python code using twikit)              │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                     TWIKIT PUBLIC API                            │
│  Client, Tweet, User, Message, Community, List, etc.             │
└────────────┬────────────────────────────┬────────────────────────┘
             │                            │
             ▼                            ▼
┌────────────────────────┐    ┌──────────────────────────────────┐
│   CLIENT LAYER         │    │   DATA MODEL LAYER               │
│ - Client class         │◄───┤ - Tweet, User, Message           │
│ - GQLClient (GraphQL)  │    │ - Community, List, Group         │
│ - V11Client (REST)     │    │ - Media, Notification, etc.      │
└────────┬───────────────┘    └──────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│                 AUTHENTICATION & SECURITY LAYER                  │
│ - Flow (multi-step auth)                                         │
│ - ClientTransaction (bot detection evasion)                      │
│ - CaptchaSolver (account unlock)                                 │
│ - UIMetrics (browser simulation)                                 │
└────────┬────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│                      UTILITY LAYER                               │
│ - Result[T] (pagination)                                         │
│ - find_dict() (data extraction)                                  │
│ - build_tweet_data(), build_user_data() (normalization)          │
│ - Constants (endpoints, feature flags)                           │
│ - Error handling (exception hierarchy)                           │
└────────┬────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│                       HTTP LAYER                                 │
│                   httpx.AsyncClient                              │
│            (async HTTP with proxy support)                       │
└────────┬────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    TWITTER/X API                                 │
│          GraphQL + REST API v1.1 Endpoints                       │
└─────────────────────────────────────────────────────────────────┘
```

### Architectural Layers

#### Layer 1: User Application
- Developer's Python code using Twikit
- Imports: `from twikit import Client, Tweet, User`
- All interactions are asynchronous (`await client.login(...)`)

#### Layer 2: Public API (Domain Interface)
- **Exported in `__init__.py`**: Client, Tweet, User, Message, etc.
- Provides high-level abstractions for Twitter concepts
- Hides complexity of API interactions

#### Layer 3: Client Layer (API Orchestration)
- **Client**: Main entry point, handles authentication, request routing
- **GQLClient**: GraphQL endpoint manager
- **V11Client**: REST API v1.1 endpoint manager
- Manages cookies, sessions, headers, error handling

#### Layer 4: Authentication & Security
- **Flow**: Multi-step authentication state machine
- **ClientTransaction**: Generates anti-bot transaction IDs
- **CaptchaSolver**: Automatic captcha solving (Arkose/Capsolver)
- **UIMetrics**: Browser behavior simulation

#### Layer 5: Utility Layer
- **Result[T]**: Generic pagination container
- Data parsing functions (JSON → Python objects)
- Query builders for advanced search
- Constants (feature flags, endpoints)

#### Layer 6: HTTP Layer
- **httpx**: Async HTTP library
- Connection pooling, proxy support
- Request/response handling

#### Layer 7: Twitter API
- GraphQL endpoints (primary)
- REST API v1.1 (legacy features)

---

## 3. Layer 2: File-Based Organization

### Directory Structure

```
twikit/
├── __init__.py                   # Public API exports
│
├── client/                       # API Client Implementation
│   ├── __init__.py
│   ├── client.py                 # Main Client class (4324 lines, 114 methods)
│   ├── gql.py                    # GraphQL endpoint definitions (~100 endpoints)
│   └── v11.py                    # REST API v1.1 endpoint definitions
│
├── guest/                        # Guest (unauthenticated) client variant
│   ├── __init__.py
│   ├── client.py                 # GuestClient class
│   ├── tweet.py                  # Guest tweet data model
│   └── user.py                   # Guest user data model
│
├── _captcha/                     # Captcha solving functionality
│   ├── __init__.py
│   ├── base.py                   # CaptchaSolver base class
│   └── capsolver.py              # Capsolver API integration
│
├── x_client_transaction/         # Bot detection evasion
│   ├── __init__.py
│   ├── transaction.py            # ClientTransaction class
│   ├── cubic_curve.py            # Cubic Bezier curve calculations
│   ├── interpolate.py            # Animation interpolation
│   ├── rotation.py               # Rotation matrix transformations
│   └── utils.py                  # Transaction utilities
│
├── ui_metrics/                   # UI metrics handling
│   ├── __init__.py
│   └── dom.py                    # DOM metrics solving
│
├── Data Model Files (Entities)
│   ├── tweet.py                  # Tweet class (2000+ lines)
│   ├── user.py                   # User class
│   ├── message.py                # Message (Direct Message) class
│   ├── community.py              # Community, CommunityMember, CommunityCreator
│   ├── list.py                   # List class
│   ├── group.py                  # Group (group DM) class
│   ├── notification.py           # Notification class
│   ├── trend.py                  # Trend, PlaceTrend, Location classes
│   ├── media.py                  # Media classes (Photo, Video, AnimatedGif)
│   ├── bookmark.py               # BookmarkFolder class
│   ├── geo.py                    # Place (geographic location) class
│   └── streaming.py              # StreamingSession, Payload classes
│
├── Core Utility Files
│   ├── utils.py                  # Result[T], Flow, data parsing, query building
│   ├── constants.py              # GraphQL features, tokens, domains (2000+ lines)
│   ├── errors.py                 # Exception hierarchy
│   └── py.typed                  # PEP 561 type hint marker
│
└── Configuration
    └── cookies_file.json         # Persisted authentication cookies (runtime)
```

### File Responsibilities

#### `__init__.py` (Public API)
**Purpose**: Define the public interface of the library

**Exports**:
```python
# Core client
from .client.client import Client

# Entity models
from .tweet import Tweet, Poll, ScheduledTweet, CommunityNote
from .user import User
from .message import Message
from .community import Community, CommunityMember, CommunityCreator, CommunityRule
from .list import List
from .group import Group, GroupMessage
from .trend import Trend
from .geo import Place
from .notification import Notification
from .bookmark import BookmarkFolder

# Utilities
from .utils import build_query
from .errors import *  # All exception classes
from ._captcha import Capsolver
```

**Version**: `__version__ = '2.3.3'`

**Platform-Specific Setup**:
```python
if os.name == 'nt':  # Windows
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
```

---

#### `client/client.py` (Main Client)
**Size**: 4324 lines, 114 async methods

**Purpose**: Central orchestrator for all Twitter API interactions

**Key Sections**:
1. **Initialization (lines 92-150)**: Setup HTTP client, language, proxy, captcha solver
2. **Authentication Methods (lines 150-500)**: login, logout, unlock, delegate accounts
3. **Tweet Operations (lines 500-1200)**: search, create, delete, favorite, retweet
4. **User Operations (lines 1200-1800)**: get users, follow, block, mute
5. **Timeline Methods (lines 1800-2000)**: get_timeline, get_home_timeline
6. **Direct Messaging (lines 2000-2500)**: send_dm, get_dm_history, reactions
7. **Lists (lines 2500-2800)**: create_list, add_member, get_list_tweets
8. **Communities (lines 2800-3200)**: join, leave, get_community_tweets
9. **Bookmarks (lines 3200-3500)**: bookmark, unbookmark, folders
10. **Media Upload (lines 3500-3800)**: upload_media, check_status
11. **Notifications (lines 3800-4000)**: get_notifications
12. **Streaming (lines 4000-4200)**: get_streaming_session
13. **Internal Helpers (lines 4200-4324)**: _request, error handling

**Dependencies**:
```python
from httpx import AsyncClient, AsyncHTTPTransport
from .gql import GQLClient
from .v11 import V11Client
from ..x_client_transaction import ClientTransaction
from .._captcha import Capsolver
# + all entity models
# + all utility functions
```

---

#### `client/gql.py` (GraphQL Endpoints)
**Purpose**: Define all GraphQL endpoint URLs and query structures

**Structure**:
```python
class Endpoint:
    """GraphQL endpoint definitions"""

    # Tweet operations
    SEARCH_TIMELINE = 'flaR-PUMshxFWZWPNpq4zA/SearchTimeline'
    CREATE_TWEET = 'SiM_cAu83R0wnrpmKQQSEw/CreateTweet'
    DELETE_TWEET = 'VaenaVgh5q5ih7kvyVjgtg/DeleteTweet'
    FAVORITE_TWEET = 'lI07N6Otwv1PhnEgXILM7A/FavoriteTweet'
    UNFAVORITE_TWEET = 'ZYKSe-w7KEslx3JhSIk5LA/UnfavoriteTweet'

    # User operations
    USER_BY_SCREEN_NAME = 'NimuplG1OB7Fd2btCLdBOw/UserByScreenName'
    USER_BY_REST_ID = 'tD8zKvQzwY3kdx5yz6YmOw/UserByRestId'
    FOLLOW_USER = 'iorA6W8-JC5F7p9N6qg5Dg/FollowUser'
    UNFOLLOW_USER = 'p4i3i5Gxg8mYLz8j8g4Lzg/UnfollowUser'

    # Timeline operations
    HOME_TIMELINE = 'U0cdisy7QFIoTfu3-Okw0A/HomeTimeline'
    HOME_LATEST_TIMELINE = 'U0cdisy7QFIoTfu3-Okw0A/HomeLatestTimeline'
    USER_TWEETS = 'V1ze5q7RgPSJkQ8i9nBD2A/UserTweets'

    # ... ~100+ more endpoints
```

**Endpoint Format**:
```
https://x.com/i/api/graphql/{endpoint_id}/{operation_name}
```

**Example**:
```
https://x.com/i/api/graphql/NimuplG1OB7Fd2btCLdBOw/UserByScreenName
```

---

#### `client/v11.py` (REST API v1.1)
**Purpose**: Define legacy REST API v1.1 endpoints

**Endpoints**:
```python
class V11Endpoint:
    # Authentication
    GUEST_ACTIVATE = 'https://api.x.com/1.1/guest/activate.json'
    ONBOARDING_TASK = 'https://api.x.com/1.1/onboarding/task.json'

    # Media
    MEDIA_UPLOAD = 'https://upload.x.com/1.1/media/upload.json'
    MEDIA_METADATA_CREATE = 'https://upload.x.com/1.1/media/metadata/create.json'

    # Trends
    TRENDS_AVAILABLE = 'https://api.x.com/1.1/trends/available.json'
    TRENDS_PLACE = 'https://api.x.com/1.1/trends/place.json'

    # Direct Messages
    DM_NEW = 'https://api.x.com/1.1/direct_messages/events/new.json'
    DM_DESTROY = 'https://api.x.com/1.1/direct_messages/events/destroy.json'
```

**Why v1.1 Still Used**:
- Guest token activation (required for unauthenticated access)
- Media uploads (not migrated to GraphQL)
- Some authentication flows
- Trends API (no GraphQL equivalent)

---

#### `tweet.py` (Tweet Model)
**Size**: 2000+ lines

**Purpose**: Represent a single tweet with all its properties and methods

**Structure**:
1. **Class Definition & Init (lines 1-110)**
2. **Properties (lines 111-500)**: id, text, created_at, user, media, counts, etc.
3. **Relationship Properties (lines 500-700)**: quote, retweeted_tweet, reply_to
4. **Media Properties (lines 700-900)**: media, photos, videos, gifs
5. **Metadata Properties (lines 900-1100)**: hashtags, urls, mentions, card
6. **Action Methods (lines 1100-1500)**: favorite(), retweet(), reply(), delete()
7. **Lazy Loading Methods (lines 1500-1800)**: get_replies(), get_related_tweets()
8. **Community Notes (lines 1800-2000)**: community_note property

**Key Properties** (100+ total):
```python
@property
def id(self) -> str:
    return self._data['rest_id']

@property
def text(self) -> str:
    return self._legacy['full_text']

@property
def user(self) -> User:
    if self.user is None:
        user_data = find_dict(self._data, 'core')[0]['user_results']['result']
        self.user = User(self._client, user_data)
    return self.user

@property
def media(self) -> list[Photo | Video | AnimatedGif]:
    entities = self._legacy.get('extended_entities', {})
    media_list = entities.get('media', [])
    return [_media_from_data(m) for m in media_list]
```

**Action Methods**:
```python
async def favorite(self) -> Tweet:
    """Like this tweet"""
    return await self._client.favorite_tweet(self.id)

async def retweet(self) -> Tweet:
    """Retweet this tweet"""
    return await self._client.create_retweet(self.id)

async def reply(self, text: str, **kwargs) -> Tweet:
    """Reply to this tweet"""
    return await self._client.create_tweet(
        text=text,
        reply_to=self.id,
        **kwargs
    )
```

---

#### `user.py` (User Model)
**Purpose**: Represent a Twitter user with profile data and methods

**Structure**:
1. **Initialization (lines 89-126)**: Parse user data from API response
2. **Properties (lines 127-130)**: created_at_datetime
3. **Tweet Methods (lines 131-200)**: get_tweets(), get_timeline()
4. **Social Methods (lines 200-350)**: follow(), unfollow(), block(), mute()
5. **Relationship Methods (lines 350-450)**: get_followers(), get_following()
6. **Messaging Methods (lines 450-500)**: send_dm()

**Data Attributes** (50+ total):
```python
def __init__(self, client: Client, data: dict):
    self._client = client
    legacy = data['legacy']

    # Identity
    self.id: str = data['rest_id']
    self.name: str = legacy['name']
    self.screen_name: str = legacy['screen_name']

    # Profile
    self.profile_image_url: str = legacy['profile_image_url_https']
    self.profile_banner_url: str = legacy.get('profile_banner_url')
    self.description: str = legacy['description']
    self.location: str = legacy['location']

    # Counts
    self.followers_count: int = legacy['followers_count']
    self.following_count: int = legacy['friends_count']
    self.statuses_count: int = legacy['statuses_count']

    # Verification
    self.is_blue_verified: bool = data['is_blue_verified']
    self.verified: bool = legacy['verified']
```

**Delegation Pattern**:
```python
async def follow(self) -> User:
    """Follow this user (delegates to client)"""
    return await self._client.follow_user(self.id)

async def get_tweets(
    self,
    tweet_type: Literal['Tweets', 'Replies', 'Media', 'Likes'],
    count: int = 40
) -> Result[Tweet]:
    """Get user's tweets (delegates to client)"""
    return await self._client.get_user_tweets(
        self.id, tweet_type, count
    )
```

---

#### `utils.py` (Utility Functions)
**Purpose**: Provide reusable utilities for data handling and pagination

**Key Components**:

**1. Result[T] - Generic Pagination Container**
```python
class Result(Generic[T]):
    """Paginated result container with cursor-based navigation"""

    def __init__(
        self,
        results: list[T],
        fetch_next_result: Awaitable | None = None,
        next_cursor: str | None = None,
        fetch_previous_result: Awaitable | None = None,
        previous_cursor: str | None = None
    ):
        self.__results = results
        self.next_cursor = next_cursor
        self.__fetch_next_result = fetch_next_result
        self.previous_cursor = previous_cursor
        self.__fetch_previous_result = fetch_previous_result

    async def next(self) -> Result[T]:
        """Fetch next page"""
        if self.__fetch_next_result is None:
            return Result([])
        return await self.__fetch_next_result()

    def __iter__(self):
        yield from self.__results

    def __getitem__(self, index: int) -> T:
        return self.__results[index]
```

**2. Flow - Authentication State Machine**
```python
class Flow:
    """Handles multi-step authentication flows"""

    def __init__(self, client: Client, guest_token: str):
        self._client = client
        self.guest_token = guest_token
        self.response = None

    async def execute_task(self, *subtask_inputs, **kwargs):
        """Execute next step in authentication flow"""
        response, _ = await self._client.v11.onboarding_task(
            self.guest_token,
            self.token,
            list(subtask_inputs),
            **kwargs
        )
        self.response = response

    @property
    def token(self) -> str | None:
        """Get flow token for next step"""
        if self.response is None:
            return None
        return self.response.get('flow_token')
```

**3. find_dict() - Nested Dictionary Traversal**
```python
def find_dict(obj: list | dict, key: str | int, find_one: bool = False) -> list[Any]:
    """
    Recursively search for a key in nested dictionaries/lists.

    Example:
        data = {'a': {'b': {'c': 'value'}}}
        find_dict(data, 'c')  # Returns ['value']
    """
    results = []
    if isinstance(obj, dict):
        if key in obj:
            results.append(obj.get(key))
            if find_one:
                return results

    if isinstance(obj, (list, dict)):
        for elem in (obj if isinstance(obj, list) else obj.values()):
            r = find_dict(elem, key, find_one)
            results += r
            if r and find_one:
                return results
    return results
```

**4. build_tweet_data() - Response Normalization**
```python
def build_tweet_data(tweet: dict) -> dict:
    """
    Normalize tweet data from different API responses into standard format.

    Different endpoints return slightly different structures:
    - Some have 'rest_id', others just 'id'
    - Some have nested 'legacy', others have flat structure

    This function standardizes all variations.
    """
    if tweet.get('__typename') == 'TweetWithVisibilityResults':
        tweet = tweet['tweet']

    result = tweet.get('result', tweet)

    # Normalize to standard format
    return {
        'rest_id': result.get('rest_id') or str(result['id']),
        'legacy': result.get('legacy', result),
        'core': result.get('core'),
        'views': result.get('views'),
        # ... more normalization
    }
```

**5. build_query() - Advanced Search Query Builder**
```python
def build_query(
    query: str,
    filters: list[SearchQueryFilter],
    exclude: list[SearchQueryExclude]
) -> str:
    """
    Build Twitter advanced search query string.

    Example:
        build_query(
            'python',
            [('from', 'user123'), ('min_faves', '10')],
            [('mentions', 'spam_bot')]
        )
        # Returns: 'python from:user123 min_faves:10 -mentions:spam_bot'
    """
    for filter_type, filter_value in filters:
        query += f' {filter_type}:{filter_value}'

    for exclude_type, exclude_value in exclude:
        query += f' -{exclude_type}:{exclude_value}'

    return query
```

---

#### `constants.py` (Configuration)
**Size**: 2000+ lines

**Purpose**: Centralize all configuration, feature flags, and constants

**Key Constants**:

**1. Bearer Token**
```python
TOKEN = 'AAAAAAAAAAAAAAAAAAAAANRILgAAAAAAnNwIzUejRCOuH5E6I8xnZz4puTs%3D1Zv7ttfk8LF81IUq16cHjhLTvJu4FA33AGWWjCpTnA'
```
This is Twitter's web app bearer token (public, extracted from JS).

**2. Domains**
```python
DOMAIN = 'x.com'
X_DOMAIN = 'api.x.com'
UPLOAD_DOMAIN = 'upload.x.com'
```

**3. GraphQL Feature Flags** (100+ flags)
```python
FEATURES = {
    # Core features
    'responsive_web_graphql_timeline_navigation_enabled': True,
    'unified_cards_ad_metadata_container_dynamic_card_content_query_enabled': True,

    # Tweet features
    'responsive_web_edit_tweet_api_enabled': True,
    'view_counts_everywhere_api_enabled': True,
    'rweb_video_timestamps_enabled': True,
    'tweetypie_unmention_optimization_enabled': True,

    # User features
    'responsive_web_twitter_blue_verified_badge_is_enabled': True,
    'verified_phone_label_enabled': True,

    # Community features
    'communities_web_enable_tweet_community_results_fetch': True,
    'c9s_tweet_anatomy_moderator_badge_enabled': True,

    # Media features
    'responsive_web_media_download_video_enabled': True,
    'responsive_web_enhance_cards_enabled': True,

    # ... 100+ more feature flags
}
```

These flags tell Twitter what data to include in responses. They're extracted from Twitter's web app JavaScript and enable/disable specific API features.

**4. Field Flags** (for GraphQL queries)
```python
FIELD_TOGGLES = {
    'withArticleRichContentState': True,
    'withArticlePlainText': False,
    'withGrokAnalyze': False,
    'withDisallowedReplyControls': True,
}
```

---

#### `errors.py` (Exception Hierarchy)
**Purpose**: Define all custom exceptions with proper inheritance

**Exception Hierarchy**:
```python
class TwitterException(Exception):
    """Base exception for all Twitter API errors"""
    pass

# HTTP Status-Based Exceptions
class BadRequest(TwitterException):              # 400
    pass

class Unauthorized(TwitterException):            # 401
    pass

class Forbidden(TwitterException):               # 403
    pass

class NotFound(TwitterException):                # 404
    pass

class RequestTimeout(TwitterException):          # 408
    pass

class TooManyRequests(TwitterException):         # 429
    pass

class ServerError(TwitterException):             # 500+
    pass

# Business Logic Exceptions
class CouldNotTweet(TwitterException):
    """Tweet creation failed (generic)"""
    pass

class DuplicateTweet(CouldNotTweet):
    """Tweet text is duplicate of recent tweet"""
    pass

class TweetNotAvailable(TwitterException):
    """Tweet deleted or visibility restricted"""
    pass

class InvalidMedia(TwitterException):
    """Media upload failed or invalid format"""
    pass

class UserNotFound(TwitterException):
    """User doesn't exist or is suspended"""
    pass

class UserUnavailable(TwitterException):
    """User is protected or blocked you"""
    pass

class AccountSuspended(TwitterException):
    """Account is suspended"""
    pass

class AccountLocked(TwitterException):
    """Account is locked, requires unlock flow"""
    pass
```

**Error Code Mapping**:
```python
ERROR_CODE_TO_EXCEPTION = {
    37: AccountSuspended,      # Account suspended
    64: AccountSuspended,      # Account suspended (variant)
    187: DuplicateTweet,       # Duplicate status
    324: InvalidMedia,         # Media upload failed
    326: AccountLocked,        # Account locked (triggers auto-unlock)
}
```

**Error Handling Function**:
```python
def raise_exceptions_from_response(
    status_code: int,
    error_code: int | None = None
) -> None:
    """
    Raise appropriate exception based on HTTP status and error code.

    Usage in client:
        response = await self.http.post(url, ...)
        if response.status_code != 200:
            raise_exceptions_from_response(
                response.status_code,
                response.json().get('errors', [{}])[0].get('code')
            )
    """
    if error_code in ERROR_CODE_TO_EXCEPTION:
        raise ERROR_CODE_TO_EXCEPTION[error_code]()

    if status_code == 400:
        raise BadRequest()
    elif status_code == 401:
        raise Unauthorized()
    # ... etc
```

---

#### `x_client_transaction/transaction.py` (Bot Detection Evasion)
**Purpose**: Generate cryptographic `X-Client-Transaction-Id` headers to bypass bot detection

**Background**: Twitter requires a special header `X-Client-Transaction-Id` that proves the request comes from a real browser, not a bot. This is generated from:
1. Animation keyframes extracted from CSS
2. Cryptographic keys from HTML meta tags
3. Dynamic JavaScript file parsing

**Class Structure**:
```python
class ClientTransaction:
    """
    Generates X-Client-Transaction-Id headers for bot detection evasion.

    Process:
    1. Fetch Twitter homepage HTML
    2. Extract verification key from meta tag
    3. Fetch on-demand JavaScript file
    4. Parse CSS animation keyframes
    5. Extract byte indices from animations
    6. Generate transaction ID from request path + key bytes
    """

    def __init__(self):
        self.home_page_response: Response | None = None
        self.key: str = None              # Base64 verification key
        self.key_bytes: list[int] = None  # Decoded key bytes
        self.animation_key: str = None    # Computed animation key

    async def init(
        self,
        session: AsyncClient,
        headers: dict,
        home_page_response: Response = None
    ):
        """
        Initialize transaction generator.

        Steps:
        1. Get homepage (or use cached response)
        2. Extract meta[name='twitter-site-verification'] content
        3. Parse on-demand JS file URL from script tags
        4. Fetch JS file and extract key computation logic
        5. Extract CSS animation keyframes
        6. Compute animation key from keyframes
        """
        # Get homepage
        if home_page_response is None:
            home_page_response = await session.get('https://x.com/')

        self.home_page_response = home_page_response

        # Extract verification key
        soup = BeautifulSoup(home_page_response.text, 'lxml')
        meta_tag = soup.find('meta', {'name': 'twitter-site-verification'})
        self.key = meta_tag['content']
        self.key_bytes = list(base64.b64decode(self.key))

        # Extract and parse JavaScript
        await self._extract_animation_key(session, headers)

    def generate_transaction_id(
        self,
        method: str,
        path: str
    ) -> str:
        """
        Generate transaction ID for a specific request.

        Args:
            method: HTTP method (GET, POST, etc.)
            path: Request path (e.g., '/i/api/graphql/...')

        Returns:
            Base64-encoded transaction ID

        Algorithm:
        1. Concatenate method + path
        2. Extract byte indices from animation key
        3. XOR key bytes with path hash
        4. Base64 encode result
        """
        if self.animation_key is None:
            raise ValueError('ClientTransaction not initialized')

        # Extract indices from animation key
        indices = self._parse_animation_indices(self.animation_key)

        # Build transaction bytes
        transaction_bytes = []
        for idx in indices:
            byte_val = self.key_bytes[idx]
            transaction_bytes.append(byte_val)

        # Hash with request path
        path_hash = hashlib.md5(f'{method}:{path}'.encode()).digest()

        # XOR operation
        result = bytes([
            transaction_bytes[i] ^ path_hash[i % len(path_hash)]
            for i in range(len(transaction_bytes))
        ])

        # Base64 encode
        return base64.b64encode(result).decode()
```

**Supporting Modules**:

**cubic_curve.py**: Cubic Bezier curve calculations
```python
def cubic_bezier_at_time(
    t: float,
    p0: float,
    p1: float,
    p2: float,
    p3: float
) -> float:
    """
    Calculate point on cubic Bezier curve at time t.

    Used to interpolate CSS animation values.
    Bezier curves defined by 4 control points.
    """
    return (
        (1 - t)**3 * p0 +
        3 * (1 - t)**2 * t * p1 +
        3 * (1 - t) * t**2 * p2 +
        t**3 * p3
    )
```

**rotation.py**: Matrix transformations
```python
def rotation_matrix_3d(
    angle_x: float,
    angle_y: float,
    angle_z: float
) -> list[list[float]]:
    """3D rotation matrix for CSS transforms"""
    # ... matrix math
```

**interpolate.py**: Animation interpolation
```python
def interpolate_color(
    color1: str,
    color2: str,
    t: float
) -> str:
    """Interpolate between two colors at time t"""
    # Parse hex colors
    r1, g1, b1 = parse_hex_color(color1)
    r2, g2, b2 = parse_hex_color(color2)

    # Linear interpolation
    r = int(r1 + (r2 - r1) * t)
    g = int(g1 + (g2 - g1) * t)
    b = int(b1 + (b2 - b1) * t)

    return f'#{r:02x}{g:02x}{b:02x}'
```

---

#### `_captcha/base.py` (Captcha Solver)
**Purpose**: Automatically unlock locked accounts using Capsolver service

**Usage Scenario**:
- Twitter locks account after suspicious activity
- Returns error code 326
- Client automatically solves captcha and unlocks
- Retries original request

**Class Structure**:
```python
class CaptchaSolver:
    """
    Solves Arkose captcha challenges to unlock accounts.

    Integration with Capsolver service (capsolver.com).
    """

    def __init__(self, api_key: str):
        self.api_key = api_key
        self.capsolver_url = 'https://api.capsolver.com/createTask'

    async def solve(
        self,
        unlock_html: str,
        client: AsyncClient
    ) -> dict:
        """
        Solve captcha from unlock page HTML.

        Process:
        1. Parse unlock HTML for challenge data
        2. Extract authenticity_token and assignment_token
        3. Send to Capsolver API
        4. Poll for solution
        5. Submit solution to Twitter
        6. Return verification result
        """
        # Parse unlock page
        soup = BeautifulSoup(unlock_html, 'lxml')

        # Extract tokens
        auth_token = soup.find('input', {'name': 'authenticity_token'})['value']
        assignment_token = soup.find('input', {'name': 'assignment_token'})['value']

        # Create Capsolver task
        task_data = {
            'clientKey': self.api_key,
            'task': {
                'type': 'FunCaptchaTaskProxyLess',
                'websiteURL': 'https://x.com/',
                'websitePublicKey': '...',  # Arkose public key
                'data': assignment_token
            }
        }

        # Submit to Capsolver
        response = await client.post(self.capsolver_url, json=task_data)
        task_id = response.json()['taskId']

        # Poll for solution
        solution = await self._poll_solution(task_id, client)

        # Submit to Twitter unlock endpoint
        unlock_response = await client.post(
            'https://x.com/account/access',
            data={
                'authenticity_token': auth_token,
                'assignment_token': assignment_token,
                'verification_string': solution,
                'language_code': 'en'
            }
        )

        return unlock_response
```

---

## 4. Layer 3: Class-Based Architecture

### Class Hierarchy & Relationships

```
┌────────────────────────────────────────────────────────────────┐
│                        httpx.AsyncClient                        │
│                    (Third-party HTTP library)                   │
└───────────────────────────┬────────────────────────────────────┘
                            │ inherits
                            ▼
┌────────────────────────────────────────────────────────────────┐
│                      twikit.Client                              │
│  - http: AsyncClient                                            │
│  - gql: GQLClient                                               │
│  - v11: V11Client                                               │
│  - client_transaction: ClientTransaction                        │
│  - captcha_solver: CaptchaSolver | None                         │
│  + 114 async methods                                            │
└───────────┬────────────────────────────────────────────────────┘
            │ referenced by (composition)
            ▼
┌────────────────────────────────────────────────────────────────┐
│                    Entity Model Classes                         │
│             (All contain _client: Client reference)             │
│                                                                  │
│  Tweet                  User                  Message            │
│  Community              List                  Group              │
│  Notification           Trend                 BookmarkFolder     │
│  Media (abstract)       Place                 StreamingSession   │
│    ├─ Photo                                                      │
│    ├─ Video                                                      │
│    └─ AnimatedGif                                                │
└────────────────────────────────────────────────────────────────┘

Supporting Classes:
┌────────────────────────────────────────────────────────────────┐
│ Result[T] (Generic)            │ Flow                           │
│ - Pagination container         │ - Auth state machine           │
│ - Cursor-based navigation      │ - Multi-step flows             │
└────────────────────────────────┴────────────────────────────────┘

┌────────────────────────────────────────────────────────────────┐
│ ClientTransaction              │ CaptchaSolver                  │
│ - Bot detection evasion        │ - Account unlock               │
│ - Transaction ID generation    │ - Arkose challenge solving     │
└────────────────────────────────┴────────────────────────────────┘

NamedTuple Classes (Immutable Data):
┌────────────────────────────────────────────────────────────────┐
│ CommunityCreator, CommunityRule, Location                       │
└────────────────────────────────────────────────────────────────┘
```

### Core Class Details

#### Client Class

**Inheritance**: No inheritance (composition-based)

**Attributes**:
```python
class Client:
    # HTTP
    http: AsyncClient              # httpx async client

    # API clients
    gql: GQLClient                 # GraphQL endpoint manager
    v11: V11Client                 # REST API v1.1 manager

    # Authentication
    _guest_token: str | None       # Guest token (unauthenticated)
    _csrf_token: str | None        # CSRF token from cookies
    _user_id: str | None           # Logged-in user ID

    # Security
    client_transaction: ClientTransaction  # Bot detection
    captcha_solver: Capsolver | None       # Account unlock

    # Configuration
    _language: str                 # API language (default: 'en-US')
    _proxy: str | None             # Proxy URL
    _user_agent: str               # User agent string
```

**Method Categories** (114 total async methods):

1. **Authentication (6)**:
   - `login()`, `logout()`, `unlock()`
   - `user_id` (property), `user()` (get current user)
   - `set_delegate_account()`

2. **Tweet Operations (20)**:
   - `search_tweet()`, `search_similar_tweets()`
   - `create_tweet()`, `delete_tweet()`
   - `get_tweet_by_id()`, `get_tweets_by_ids()`
   - `favorite_tweet()`, `unfavorite_tweet()`
   - `create_retweet()`, `delete_retweet()`
   - `create_scheduled_tweet()`, `get_scheduled_tweets()`
   - `vote()` (poll voting)

3. **User Operations (18)**:
   - `get_user_by_id()`, `get_user_by_screen_name()`
   - `get_users_by_ids()`, `search_user()`
   - `follow_user()`, `unfollow_user()`
   - `block_user()`, `unblock_user()`, `get_blocked_users()`
   - `mute_user()`, `unmute_user()`, `get_muted_users()`
   - `get_user_followers()`, `get_user_following()`
   - `get_user_tweets()`, `get_user_likes()`
   - `get_user_verified_followers()`

4. **Timeline Operations (3)**:
   - `get_timeline()`, `get_latest_timeline()`, `get_home_timeline()`

5. **Direct Messaging (12)**:
   - `send_dm()`, `get_dm_history()`, `delete_dm()`
   - `send_dm_to_group()`, `get_group_dm_history()`
   - `create_group()`, `edit_group()`, `add_members_to_group()`
   - `add_reaction_to_message()`, `remove_reaction_from_message()`

6. **Lists (13)**:
   - `create_list()`, `edit_list()`, `delete_list()`
   - `pin_list()`, `unpin_list()`
   - `get_list()`, `get_lists()`
   - `add_list_member()`, `remove_list_member()`
   - `get_list_tweets()`, `get_list_members()`, `get_list_subscribers()`
   - `subscribe_list()`, `unsubscribe_list()`

7. **Bookmarks (10)**:
   - `bookmark_tweet()`, `unbookmark_tweet()`
   - `get_bookmarks()`, `delete_all_bookmarks()`
   - `create_bookmark_folder()`, `edit_bookmark_folder()`, `delete_bookmark_folder()`
   - `add_bookmark_to_folder()`, `remove_bookmark_from_folder()`

8. **Communities (12)**:
   - `search_community()`, `get_community()`
   - `join_community()`, `leave_community()`, `request_to_join_community()`
   - `get_community_tweets()`, `get_community_members()`
   - `get_community_moderators()`, `get_community_rules()`

9. **Media (3)**:
   - `upload_media()`, `check_media_status()`, `create_media_metadata()`

10. **Notifications (1)**:
    - `get_notifications()`

11. **Trends (3)**:
    - `get_trends()`, `get_available_locations()`, `get_place_trends()`

12. **Streaming (2)**:
    - `get_streaming_session()`, `_stream()` (internal)

13. **Internal Helpers (15+)**:
    - `request()` - Main HTTP request wrapper
    - `_get_guest_token()` - Obtain guest token
    - `_validate_session()` - Check if logged in
    - Error handling, cookie management, etc.

**Key Method: request()**
```python
async def request(
    self,
    method: str,
    url: str,
    auto_unlock: bool = True,
    **kwargs
) -> Response:
    """
    Core HTTP request method with automatic error handling.

    Features:
    - Adds authentication headers (bearer token, CSRF)
    - Validates client transaction ID
    - Handles account locks (auto-unlock if captcha_solver provided)
    - Raises appropriate exceptions on errors
    - Logs requests for debugging

    Args:
        method: HTTP method (GET, POST, etc.)
        url: Full URL or path (prepends domain if needed)
        auto_unlock: Whether to auto-unlock on 326 error
        **kwargs: Additional httpx request parameters

    Returns:
        httpx.Response object

    Raises:
        AccountLocked: If account is locked and no captcha solver
        AccountSuspended: If account is suspended
        TooManyRequests: If rate limited
        ... other TwitterException subclasses
    """
    # Validate client transaction
    if self.client_transaction is None:
        await self.client_transaction.init(self.http, self._base_headers)

    # Add transaction ID header
    headers = kwargs.get('headers', {})
    headers['X-Client-Transaction-Id'] = \
        self.client_transaction.generate_transaction_id(method, url)

    # Add CSRF token for POST requests
    if method == 'POST':
        headers['X-Csrf-Token'] = self._csrf_token

    # Add authorization
    headers['Authorization'] = f'Bearer {TOKEN}'

    kwargs['headers'] = headers

    # Make request
    response = await self.http.request(method, url, **kwargs)

    # Handle errors
    if response.status_code != 200:
        error_data = response.json()
        errors = error_data.get('errors', [])

        if errors:
            error_code = errors[0].get('code')

            # Account locked - try to unlock
            if error_code == 326 and auto_unlock and self.captcha_solver:
                await self.unlock()
                # Retry request
                return await self.request(method, url, auto_unlock=False, **kwargs)

            # Raise appropriate exception
            raise_exceptions_from_response(response.status_code, error_code)

    return response
```

---

#### Tweet Class

**Purpose**: Represent a single tweet with all metadata and operations

**Attributes** (100+ properties):
```python
class Tweet:
    # Internal
    _client: Client           # Reference to client for method calls
    _data: dict               # Raw API response data
    _legacy: dict             # Legacy data format (aliased for convenience)

    # Lazy-loaded relationships
    replies: Result[Tweet] | None
    reply_to: list[Tweet] | None
    related_tweets: list[Tweet] | None
    thread: list[Tweet] | None

    # Identity (properties accessing _data)
    @property
    def id(self) -> str

    @property
    def created_at(self) -> str

    @property
    def created_at_datetime(self) -> datetime

    # Content
    @property
    def text(self) -> str

    @property
    def lang(self) -> str

    # Author
    @property
    def user(self) -> User

    # Relationships
    @property
    def in_reply_to(self) -> str | None

    @property
    def is_quote_status(self) -> bool

    @property
    def quote(self) -> Tweet | None

    @property
    def retweeted_tweet(self) -> Tweet | None

    # Media
    @property
    def media(self) -> list[Photo | Video | AnimatedGif]

    # Engagement counts
    @property
    def reply_count(self) -> int

    @property
    def retweet_count(self) -> int

    @property
    def favorite_count(self) -> int

    @property
    def quote_count(self) -> int

    @property
    def bookmark_count(self) -> int

    @property
    def view_count(self) -> int | None

    # User interaction state
    @property
    def favorited(self) -> bool

    @property
    def bookmarked(self) -> bool

    # Metadata
    @property
    def place(self) -> Place | None

    @property
    def hashtags(self) -> list[str]

    @property
    def urls(self) -> list[dict]

    # Editability
    @property
    def is_edit_eligible(self) -> bool

    @property
    def edits_remaining(self) -> int

    @property
    def edit_tweet_ids(self) -> list[str]

    # Community Notes
    @property
    def community_note(self) -> CommunityNote | None

    # Poll
    @property
    def poll(self) -> Poll | None
```

**Methods**:
```python
# Action methods (delegate to client)
async def favorite(self) -> Tweet:
    """Like this tweet"""
    return await self._client.favorite_tweet(self.id)

async def unfavorite(self) -> Tweet:
    """Unlike this tweet"""
    return await self._client.unfavorite_tweet(self.id)

async def retweet(self) -> Tweet:
    """Retweet this tweet"""
    return await self._client.create_retweet(self.id)

async def delete_retweet(self) -> Response:
    """Delete retweet of this tweet"""
    return await self._client.delete_retweet(self.id)

async def bookmark(self) -> Response:
    """Bookmark this tweet"""
    return await self._client.bookmark_tweet(self.id)

async def delete(self) -> Response:
    """Delete this tweet (must be yours)"""
    return await self._client.delete_tweet(self.id)

async def reply(
    self,
    text: str,
    media_ids: list[str] | None = None,
    **kwargs
) -> Tweet:
    """Reply to this tweet"""
    return await self._client.create_tweet(
        text=text,
        media_ids=media_ids,
        reply_to=self.id,
        **kwargs
    )

# Lazy loading methods
async def get_replies(self, count: int = 40) -> Result[Tweet]:
    """Get replies to this tweet"""
    if self.replies is None:
        # Fetch replies from API
        self.replies = await self._client._get_tweet_replies(self.id, count)
    return self.replies

async def get_related_tweets(self) -> list[Tweet]:
    """Get related/similar tweets"""
    if self.related_tweets is None:
        self.related_tweets = await self._client.search_similar_tweets(self.id)
    return self.related_tweets
```

**Property Implementation Pattern**:
```python
@property
def media(self) -> list[Photo | Video | AnimatedGif]:
    """
    Extract media from tweet data.

    Media can be in different places depending on API response:
    - legacy.extended_entities.media (most common)
    - legacy.entities.media (fallback)

    Returns list of Media objects (Photo, Video, or AnimatedGif).
    """
    # Check extended_entities first (includes videos/GIFs)
    entities = self._legacy.get('extended_entities', {})
    media_list = entities.get('media', [])

    # Fallback to entities (photos only)
    if not media_list:
        entities = self._legacy.get('entities', {})
        media_list = entities.get('media', [])

    # Convert each media dict to appropriate Media subclass
    return [_media_from_data(media_data) for media_data in media_list]
```

---

#### User Class

**Purpose**: Represent a Twitter user with profile data and methods

**Attributes**:
```python
class User:
    _client: Client           # Reference to client

    # Identity
    id: str
    name: str
    screen_name: str

    # Profile
    profile_image_url: str
    profile_banner_url: str
    description: str
    location: str
    url: str

    # URLs
    description_urls: list
    urls: list

    # Verification
    is_blue_verified: bool
    verified: bool

    # Privacy
    protected: bool
    possibly_sensitive: bool

    # Permissions
    can_dm: bool
    can_media_tag: bool

    # Counts
    followers_count: int
    fast_followers_count: int
    normal_followers_count: int
    following_count: int
    favourites_count: int
    listed_count: int
    media_count: int
    statuses_count: int

    # Metadata
    created_at: str
    pinned_tweet_ids: list[str]
    is_translator: bool
    translator_type: str
    withheld_in_countries: list[str]
```

**Methods**:
```python
# Properties
@property
def created_at_datetime(self) -> datetime:
    return timestamp_to_datetime(self.created_at)

# Tweet retrieval
async def get_tweets(
    self,
    tweet_type: Literal['Tweets', 'Replies', 'Media', 'Likes'],
    count: int = 40
) -> Result[Tweet]:
    """Get user's tweets by type"""
    return await self._client.get_user_tweets(self.id, tweet_type, count)

# Social actions
async def follow(self) -> User:
    """Follow this user"""
    return await self._client.follow_user(self.id)

async def unfollow(self) -> User:
    """Unfollow this user"""
    return await self._client.unfollow_user(self.id)

async def block(self) -> User:
    """Block this user"""
    return await self._client.block_user(self.id)

async def unblock(self) -> User:
    """Unblock this user"""
    return await self._client.unblock_user(self.id)

async def mute(self) -> User:
    """Mute this user"""
    return await self._client.mute_user(self.id)

async def unmute(self) -> User:
    """Unmute this user"""
    return await self._client.unmute_user(self.id)

# Relationships
async def get_followers(self, count: int = 40) -> Result[User]:
    """Get user's followers"""
    return await self._client.get_user_followers(self.id, count)

async def get_following(self, count: int = 40) -> Result[User]:
    """Get users this user follows"""
    return await self._client.get_user_following(self.id, count)

# Messaging
async def send_dm(
    self,
    text: str,
    media_id: str | None = None
) -> Message:
    """Send direct message to this user"""
    return await self._client.send_dm(self.id, text, media_id)
```

---

#### Result[T] Class

**Purpose**: Generic pagination container with cursor-based navigation

**Type**: Generic class (`Generic[T]`)

**Attributes**:
```python
class Result(Generic[T]):
    __results: list[T]                        # Actual result items (private)
    next_cursor: str | None                   # Cursor for next page
    __fetch_next_result: Awaitable | None     # Async function to fetch next
    previous_cursor: str | None               # Cursor for previous page
    __fetch_previous_result: Awaitable | None # Async function to fetch previous
```

**Methods**:
```python
async def next(self) -> Result[T]:
    """
    Fetch next page of results.

    Returns empty Result if no more pages.
    """
    if self.__fetch_next_result is None:
        return Result([])
    return await self.__fetch_next_result()

async def previous(self) -> Result[T]:
    """
    Fetch previous page of results.

    Returns empty Result if no previous page.
    """
    if self.__fetch_previous_result is None:
        return Result([])
    return await self.__fetch_previous_result()

# List-like interface
def __iter__(self) -> Iterator[T]:
    """Iterate over results"""
    yield from self.__results

def __getitem__(self, index: int) -> T:
    """Access by index: result[0]"""
    return self.__results[index]

def __len__(self) -> int:
    """Get count: len(result)"""
    return len(self.__results)

def __repr__(self) -> str:
    """String representation"""
    return self.__results.__repr__()

@classmethod
def empty(cls) -> Result[T]:
    """Create empty result"""
    return cls([])
```

**Usage Pattern**:
```python
# Client creates Result with cursor and fetch function
async def search_tweet(
    self,
    query: str,
    product: str = 'Top',
    count: int = 20,
    cursor: str | None = None
) -> Result[Tweet]:
    """Search tweets with pagination"""

    # Make API request
    response = await self.gql.search_timeline(query, product, count, cursor)

    # Parse tweets from response
    tweets = [
        tweet_from_data(self, tweet_data)
        for tweet_data in response['entries']
    ]

    # Extract next cursor
    next_cursor = response.get('cursor')

    # Create fetch function for next page
    async def fetch_next():
        return await self.search_tweet(query, product, count, next_cursor)

    # Return Result with pagination support
    return Result(
        results=tweets,
        next_cursor=next_cursor,
        fetch_next_result=fetch_next() if next_cursor else None
    )

# User uses Result
tweets = await client.search_tweet('python', 'Latest')

# Iterate
for tweet in tweets:
    print(tweet.text)

# Access by index
first_tweet = tweets[0]

# Get next page
more_tweets = await tweets.next()

# Chain pagination
for tweet in more_tweets:
    print(tweet.text)

# Check if more pages
if more_tweets.next_cursor:
    print("More results available")
```

---

#### Media Classes

**Hierarchy**:
```
Media (base class)
├── Photo
├── Video
└── AnimatedGif
```

**Base Media Class**:
```python
class Media:
    """Base class for all media types"""

    media_key: str
    media_url: str
    display_url: str
    type: str              # 'photo', 'video', or 'animated_gif'
```

**Photo Class**:
```python
class Photo(Media):
    """Image media"""

    type: str = 'photo'

    # Dimensions
    width: int
    height: int

    # URLs
    media_url: str         # Direct image URL
    display_url: str       # Display URL (may be truncated)
```

**Video Class**:
```python
class Video(Media):
    """Video media"""

    type: str = 'video'

    # Dimensions
    width: int
    height: int

    # Duration
    duration_millis: int

    # Variants (different quality/format options)
    variants: list[VideoVariant]

    # Thumbnail
    thumbnail_url: str

    # View count
    view_count: int | None
```

**VideoVariant Class**:
```python
class VideoVariant:
    """Single video encoding variant"""

    bitrate: int | None    # Bitrate in bps
    content_type: str      # 'video/mp4', 'application/x-mpegURL', etc.
    url: str               # Direct video URL
```

**AnimatedGif Class**:
```python
class AnimatedGif(Media):
    """Animated GIF (actually MP4 video)"""

    type: str = 'animated_gif'

    # Dimensions
    width: int
    height: int

    # Variants (like Video)
    variants: list[VideoVariant]

    # Thumbnail
    thumbnail_url: str
```

**Factory Function**:
```python
def _media_from_data(data: dict) -> Photo | Video | AnimatedGif:
    """
    Create appropriate Media subclass from API data.

    Twitter returns media type in 'type' field:
    - 'photo' → Photo
    - 'video' → Video
    - 'animated_gif' → AnimatedGif
    """
    media_type = data['type']

    if media_type == 'photo':
        return Photo(
            media_key=data['media_key'],
            media_url=data['media_url_https'],
            display_url=data['display_url'],
            width=data['original_info']['width'],
            height=data['original_info']['height']
        )

    elif media_type == 'video':
        return Video(
            media_key=data['media_key'],
            media_url=data.get('media_url_https', ''),
            display_url=data['display_url'],
            width=data['original_info']['width'],
            height=data['original_info']['height'],
            duration_millis=data['video_info']['duration_millis'],
            variants=[
                VideoVariant(**variant)
                for variant in data['video_info']['variants']
            ],
            thumbnail_url=data.get('media_url_https', ''),
            view_count=data.get('mediaStats', {}).get('viewCount')
        )

    elif media_type == 'animated_gif':
        return AnimatedGif(
            media_key=data['media_key'],
            media_url=data.get('media_url_https', ''),
            display_url=data['display_url'],
            width=data['original_info']['width'],
            height=data['original_info']['height'],
            variants=[
                VideoVariant(**variant)
                for variant in data['video_info']['variants']
            ],
            thumbnail_url=data.get('media_url_https', '')
        )
```

---

## 5. Layer 4: Function-Based Implementation

This layer examines key functions and their implementations in detail.

### Authentication Flow Functions

#### `login()` - Multi-Step Authentication

**Location**: `client/client.py:150-350`

**Signature**:
```python
async def login(
    self,
    auth_info_1: str,
    auth_info_2: str | None = None,
    password: str | None = None,
    totp_secret: str | None = None,
    cookies_file: str | None = None
) -> dict
```

**Implementation**:
```python
async def login(
    self,
    auth_info_1: str,
    auth_info_2: str | None = None,
    password: str | None = None,
    totp_secret: str | None = None,
    cookies_file: str | None = None
) -> dict:
    """
    Authenticate with Twitter using username/email and password.

    Multi-step process:
    1. Get guest token
    2. Initialize SSO flow
    3. Enter username/email
    4. Enter password
    5. Handle 2FA if needed (TOTP)
    6. Handle account unlock if locked
    7. Save cookies for session persistence

    Args:
        auth_info_1: Username, email, or phone number
        auth_info_2: Email (optional, required for some accounts)
        password: Account password
        totp_secret: TOTP secret for 2FA (optional)
        cookies_file: Path to save/load cookies (optional)

    Returns:
        dict: Authentication response data
    """

    # Try to load existing cookies
    if cookies_file and os.path.exists(cookies_file):
        with open(cookies_file, 'r') as f:
            cookies = json.load(f)
            self.http.cookies.update(cookies)

            # Validate cookies
            try:
                await self._validate_session()
                return {'status': 'loaded_from_cookies'}
            except Unauthorized:
                # Cookies expired, continue with login
                pass

    # Step 1: Get guest token
    guest_token = await self._get_guest_token()

    # Step 2: Initialize authentication flow
    flow = Flow(self, guest_token)
    await flow.execute_task()  # Initial flow setup

    # Step 3: Enter username/email
    await flow.execute_task({
        'subtask_id': 'LoginEnterUserIdentifierSSO',
        'settings_list': {
            'setting_responses': [{
                'key': 'user_identifier',
                'response_data': {
                    'text_data': {'result': auth_info_1}
                }
            }],
            'link': 'next_link'
        }
    })

    # Check if additional email required
    if flow.task_id == 'LoginEnterAlternateIdentifierSubtask':
        await flow.execute_task({
            'subtask_id': flow.task_id,
            'enter_text': {
                'text': auth_info_2,
                'link': 'next_link'
            }
        })

    # Step 4: Enter password
    await flow.execute_task({
        'subtask_id': 'LoginEnterPassword',
        'enter_password': {
            'password': password,
            'link': 'next_link'
        }
    })

    # Step 5: Handle 2FA if present
    if flow.task_id == 'LoginTwoFactorAuthChallenge':
        if totp_secret is None:
            raise ValueError('2FA required but no totp_secret provided')

        # Generate TOTP code
        totp = pyotp.TOTP(totp_secret)
        code = totp.now()

        await flow.execute_task({
            'subtask_id': flow.task_id,
            'enter_text': {
                'text': code,
                'link': 'next_link'
            }
        })

    # Step 6: Handle account duplication check
    if flow.task_id == 'AccountDuplicationCheck':
        await flow.execute_task({
            'subtask_id': flow.task_id,
            'check_logged_in_account': {
                'link': 'AccountDuplicationCheck_false'
            }
        })

    # Extract cookies from response
    cookies = dict(self.http.cookies)

    # Extract CSRF token
    self._csrf_token = cookies.get('ct0')

    # Extract user ID from response
    user_data = find_dict(flow.response, 'user', find_one=True)[0]
    self._user_id = user_data['id_str']

    # Save cookies if path provided
    if cookies_file:
        with open(cookies_file, 'w') as f:
            json.dump(cookies, f)

    # Initialize client transaction for bot detection
    await self.client_transaction.init(self.http, self._base_headers)

    return flow.response
```

**Flow Visualization**:
```
login() called
    │
    ├─→ [1] Load cookies from file (if exists)
    │       └─→ Validate session
    │           ├─→ Valid: Return early
    │           └─→ Invalid: Continue
    │
    ├─→ [2] Get guest token
    │       POST /1.1/guest/activate.json
    │       Response: {"guest_token": "..."}
    │
    ├─→ [3] Initialize flow
    │       POST /1.1/onboarding/task.json
    │       Response: {"flow_token": "...", "subtasks": [...]}
    │
    ├─→ [4] Enter username
    │       POST /1.1/onboarding/task.json
    │       Body: {flow_token, subtask_id: "LoginEnterUserIdentifier", ...}
    │
    ├─→ [5] Enter email (if required)
    │       POST /1.1/onboarding/task.json
    │       Body: {flow_token, subtask_id: "LoginEnterAlternateIdentifier", ...}
    │
    ├─→ [6] Enter password
    │       POST /1.1/onboarding/task.json
    │       Body: {flow_token, subtask_id: "LoginEnterPassword", ...}
    │
    ├─→ [7] Enter 2FA code (if required)
    │       Generate TOTP code with pyotp
    │       POST /1.1/onboarding/task.json
    │       Body: {flow_token, subtask_id: "LoginTwoFactorAuthChallenge", ...}
    │
    ├─→ [8] Handle account duplication check
    │       POST /1.1/onboarding/task.json
    │
    ├─→ [9] Extract cookies and CSRF token
    │       ct0 cookie → self._csrf_token
    │       auth_token cookie → session
    │
    ├─→ [10] Save cookies to file
    │       Write JSON to cookies_file
    │
    └─→ [11] Initialize ClientTransaction
            Generate X-Client-Transaction-Id headers
```

---

#### `unlock()` - Automatic Account Unlock

**Location**: `client/client.py:400-500`

**Signature**:
```python
async def unlock(self) -> None
```

**Implementation**:
```python
async def unlock(self) -> None:
    """
    Unlock locked account using captcha solver.

    Process:
    1. Fetch unlock page HTML
    2. Extract challenge tokens
    3. Solve captcha using Capsolver
    4. Submit solution
    5. Verify unlock success

    Raises:
        ValueError: If no captcha_solver configured
        AccountLocked: If unlock fails
    """
    if self.captcha_solver is None:
        raise ValueError(
            'Account is locked but no captcha_solver configured. '
            'Visit https://x.com/account/access to unlock manually.'
        )

    # Fetch unlock page
    unlock_url = 'https://x.com/account/access'
    response = await self.http.get(unlock_url)

    # Parse HTML
    soup = BeautifulSoup(response.text, 'lxml')

    # Extract authenticity token (CSRF)
    auth_input = soup.find('input', {'name': 'authenticity_token'})
    if not auth_input:
        raise ValueError('Could not find authenticity_token in unlock page')

    authenticity_token = auth_input['value']

    # Extract assignment token (Arkose challenge ID)
    assignment_input = soup.find('input', {'name': 'assignment_token'})
    if not assignment_input:
        raise ValueError('Could not find assignment_token in unlock page')

    assignment_token = assignment_input['value']

    # Solve captcha
    solution = await self.captcha_solver.solve(
        assignment_token=assignment_token,
        website_url='https://x.com/'
    )

    # Submit solution
    submit_data = {
        'authenticity_token': authenticity_token,
        'assignment_token': assignment_token,
        'verification_string': solution,
        'language_code': 'en',
        'flow': ''
    }

    submit_response = await self.http.post(
        'https://x.com/account/access',
        data=submit_data,
        follow_redirects=False
    )

    # Check if unlock successful
    if submit_response.status_code == 302:
        # Redirect means success
        location = submit_response.headers.get('Location', '')
        if '/home' in location or '/account' in location:
            print('Account unlocked successfully')
            return

    # Parse response for error messages
    soup = BeautifulSoup(submit_response.text, 'lxml')
    error = soup.find('div', {'class': 'error-message'})
    if error:
        raise AccountLocked(f'Unlock failed: {error.text}')

    raise AccountLocked('Unlock failed with unknown error')
```

---

### Tweet Operations Functions

#### `search_tweet()` - Search with Pagination

**Location**: `client/client.py:800-900`

**Signature**:
```python
async def search_tweet(
    self,
    query: str,
    product: Literal['Top', 'Latest', 'People', 'Photos', 'Videos'] = 'Top',
    count: int = 20,
    cursor: str | None = None
) -> Result[Tweet]
```

**Implementation**:
```python
async def search_tweet(
    self,
    query: str,
    product: Literal['Top', 'Latest', 'People', 'Photos', 'Videos'] = 'Top',
    count: int = 20,
    cursor: str | None = None
) -> Result[Tweet]:
    """
    Search for tweets matching a query.

    Args:
        query: Search query (supports advanced syntax like 'from:user')
        product: Result type (Top/Latest/People/Photos/Videos)
        count: Number of results (default 20)
        cursor: Pagination cursor (for next page)

    Returns:
        Result[Tweet]: Paginated result container

    Example:
        # Basic search
        tweets = await client.search_tweet('python', 'Latest')

        # Advanced search
        tweets = await client.search_tweet(
            'python from:realpython min_faves:10',
            'Latest'
        )

        # Pagination
        more_tweets = await tweets.next()
    """

    # Build GraphQL variables
    variables = {
        'rawQuery': query,
        'count': count,
        'querySource': 'typed_query',
        'product': product
    }

    if cursor:
        variables['cursor'] = cursor

    # Make GraphQL request
    response = await self.gql._request(
        endpoint=Endpoint.SEARCH_TIMELINE,
        variables=variables,
        features=FEATURES
    )

    # Extract timeline from response
    timeline_data = find_dict(response, 'entries', find_one=True)
    if not timeline_data:
        return Result.empty()

    entries = timeline_data[0]

    # Parse tweets from entries
    tweets = []
    next_cursor = None

    for entry in entries:
        entry_id = entry.get('entryId', '')

        # Tweet entries
        if entry_id.startswith('tweet-'):
            tweet_data = find_dict(entry, 'result', find_one=True)[0]
            tweet = tweet_from_data(self, tweet_data)
            tweets.append(tweet)

        # Cursor entry (for pagination)
        elif entry_id.startswith('cursor-bottom-'):
            next_cursor = entry['content']['value']

    # Create fetch function for next page
    fetch_next = None
    if next_cursor:
        fetch_next = partial(
            self.search_tweet,
            query=query,
            product=product,
            count=count,
            cursor=next_cursor
        )()

    return Result(
        results=tweets,
        next_cursor=next_cursor,
        fetch_next_result=fetch_next
    )
```

---

#### `create_tweet()` - Create Tweet with Media

**Location**: `client/client.py:950-1100`

**Signature**:
```python
async def create_tweet(
    self,
    text: str,
    media_ids: list[str] | None = None,
    poll_options: list[str] | None = None,
    poll_duration_minutes: int = 1440,
    reply_to: str | None = None,
    conversation_control: Literal['everyone', 'followers', 'mentions'] = 'everyone',
    attachment_url: str | None = None
) -> Tweet
```

**Implementation**:
```python
async def create_tweet(
    self,
    text: str,
    media_ids: list[str] | None = None,
    poll_options: list[str] | None = None,
    poll_duration_minutes: int = 1440,
    reply_to: str | None = None,
    conversation_control: Literal['everyone', 'followers', 'mentions'] = 'everyone',
    attachment_url: str | None = None
) -> Tweet:
    """
    Create a new tweet.

    Args:
        text: Tweet text (max 280 characters, or 4000 for Twitter Blue)
        media_ids: List of media IDs from upload_media()
        poll_options: Poll options (2-4 options)
        poll_duration_minutes: Poll duration (5 to 10080 minutes)
        reply_to: Tweet ID to reply to
        conversation_control: Who can reply ('everyone', 'followers', 'mentions')
        attachment_url: URL to attach (for quote tweets)

    Returns:
        Tweet: The created tweet

    Raises:
        DuplicateTweet: If tweet is duplicate of recent tweet
        CouldNotTweet: If tweet creation fails

    Example:
        # Simple tweet
        tweet = await client.create_tweet('Hello, world!')

        # Tweet with media
        media_id = await client.upload_media('image.jpg')
        tweet = await client.create_tweet(
            'Check out this image!',
            media_ids=[media_id]
        )

        # Tweet with poll
        tweet = await client.create_tweet(
            'What is your favorite programming language?',
            poll_options=['Python', 'JavaScript', 'Rust', 'Go'],
            poll_duration_minutes=1440  # 24 hours
        )

        # Reply to tweet
        tweet = await client.create_tweet(
            'Great tweet!',
            reply_to='1234567890'
        )
    """

    # Build variables
    variables = {
        'tweet_text': text,
        'dark_request': False,
        'media': {
            'media_entities': [],
            'possibly_sensitive': False
        },
        'semantic_annotation_ids': []
    }

    # Add media
    if media_ids:
        variables['media']['media_entities'] = [
            {'media_id': media_id, 'tagged_users': []}
            for media_id in media_ids
        ]

    # Add poll
    if poll_options:
        if not (2 <= len(poll_options) <= 4):
            raise ValueError('Poll must have 2-4 options')

        variables['card_uri'] = 'card://poll'
        variables['card_data'] = {
            'choice1_label': poll_options[0],
            'choice2_label': poll_options[1],
            'duration_minutes': poll_duration_minutes
        }

        if len(poll_options) >= 3:
            variables['card_data']['choice3_label'] = poll_options[2]
        if len(poll_options) == 4:
            variables['card_data']['choice4_label'] = poll_options[3]

    # Add reply
    if reply_to:
        variables['reply'] = {
            'in_reply_to_tweet_id': reply_to,
            'exclude_reply_user_ids': []
        }

    # Add conversation control
    conversation_control_map = {
        'everyone': 'Community',
        'followers': 'ByInvitation',
        'mentions': 'ByInvitation'
    }

    variables['conversation_control'] = {
        'mode': conversation_control_map[conversation_control]
    }

    # Add attachment URL (for quote tweets)
    if attachment_url:
        variables['attachment_url'] = attachment_url

    # Make GraphQL request
    try:
        response = await self.gql._request(
            endpoint=Endpoint.CREATE_TWEET,
            variables=variables,
            features=FEATURES
        )
    except TwitterException as e:
        # Check for specific error codes
        if hasattr(e, 'error_code'):
            if e.error_code == 187:
                raise DuplicateTweet('Tweet is duplicate of recent tweet')
        raise CouldNotTweet(str(e))

    # Extract created tweet from response
    tweet_data = find_dict(
        response,
        'result',
        find_one=True
    )[0]

    # Build normalized tweet data
    tweet_data = build_tweet_data(tweet_data)

    # Create Tweet object
    tweet = tweet_from_data(self, tweet_data)

    return tweet
```

---

### Data Parsing Functions

#### `find_dict()` - Nested Dictionary Traversal

**Location**: `utils.py:111-127`

**Implementation**:
```python
def find_dict(
    obj: list | dict,
    key: str | int,
    find_one: bool = False
) -> list[Any]:
    """
    Recursively search for a key in nested dictionaries/lists.

    This is crucial for parsing Twitter API responses, which have
    deeply nested structures with varying layouts.

    Args:
        obj: Dictionary or list to search
        key: Key to search for
        find_one: Stop after finding first match

    Returns:
        list: All values found for the key

    Examples:
        # Simple nested dict
        data = {'a': {'b': {'c': 'value'}}}
        find_dict(data, 'c')  # Returns ['value']

        # Multiple occurrences
        data = {'items': [
            {'id': '1', 'name': 'A'},
            {'id': '2', 'name': 'B'}
        ]}
        find_dict(data, 'id')  # Returns ['1', '2']

        # Find first only
        find_dict(data, 'id', find_one=True)  # Returns ['1']

        # Real Twitter API usage
        response = {
            'data': {
                'user': {
                    'result': {
                        'legacy': {
                            'screen_name': 'example'
                        }
                    }
                }
            }
        }
        find_dict(response, 'screen_name')  # Returns ['example']
    """
    results = []

    # Check if current level contains key
    if isinstance(obj, dict):
        if key in obj:
            results.append(obj.get(key))
            if find_one:
                return results

    # Recursively search nested structures
    if isinstance(obj, (list, dict)):
        # Iterate over list items or dict values
        iterable = obj if isinstance(obj, list) else obj.values()

        for elem in iterable:
            r = find_dict(elem, key, find_one)
            results += r
            if r and find_one:
                return results

    return results
```

**Why This Is Important**:

Twitter's GraphQL API returns highly nested responses with inconsistent structures:

```json
{
  "data": {
    "search_by_raw_query": {
      "search_timeline": {
        "timeline": {
          "instructions": [
            {
              "type": "TimelineAddEntries",
              "entries": [
                {
                  "entryId": "tweet-1234",
                  "content": {
                    "entryType": "TimelineTimelineItem",
                    "itemContent": {
                      "itemType": "TimelineTweet",
                      "tweet_results": {
                        "result": {
                          "__typename": "Tweet",
                          "rest_id": "1234",
                          "legacy": {
                            "full_text": "Tweet text here"
                          }
                        }
                      }
                    }
                  }
                }
              ]
            }
          ]
        }
      }
    }
  }
}
```

Instead of manually navigating this with:
```python
text = response['data']['search_by_raw_query']['search_timeline']['timeline']['instructions'][0]['entries'][0]['content']['itemContent']['tweet_results']['result']['legacy']['full_text']
```

We can use:
```python
text = find_dict(response, 'full_text', find_one=True)[0]
```

---

#### `build_tweet_data()` - Response Normalization

**Location**: `utils.py:200-350`

**Implementation**:
```python
def build_tweet_data(tweet: dict) -> dict:
    """
    Normalize tweet data from different API responses.

    Twitter returns tweets in different formats depending on endpoint:
    - Some have 'rest_id', others just 'id_str'
    - Some have nested 'legacy', others are flat
    - Some wrap in 'TweetWithVisibilityResults'

    This function standardizes all variations into a consistent format.

    Args:
        tweet: Raw tweet data from API

    Returns:
        dict: Normalized tweet data with structure:
            {
                'rest_id': str,
                'legacy': dict,
                'core': dict,
                'views': dict,
                ...
            }
    """

    # Handle visibility wrapper
    if tweet.get('__typename') == 'TweetWithVisibilityResults':
        tweet = tweet['tweet']

    # Get result (sometimes wrapped)
    result = tweet.get('result', tweet)

    # Handle tombstone (deleted/unavailable tweets)
    if result.get('__typename') == 'TweetTombstone':
        return {
            'rest_id': None,
            'legacy': {},
            'tombstone': result.get('tombstone')
        }

    # Handle unavailable tweet
    if result.get('__typename') == 'TweetUnavailable':
        return {
            'rest_id': None,
            'legacy': {},
            'unavailable': True
        }

    # Extract ID (multiple possible locations)
    rest_id = (
        result.get('rest_id') or
        result.get('id_str') or
        str(result.get('id', ''))
    )

    # Extract legacy data
    legacy = result.get('legacy', {})

    # If no legacy, build from flat structure
    if not legacy:
        legacy = {
            'created_at': result.get('created_at'),
            'full_text': result.get('full_text') or result.get('text', ''),
            'favorite_count': result.get('favorite_count', 0),
            'retweet_count': result.get('retweet_count', 0),
            'reply_count': result.get('reply_count', 0),
            # ... map all fields
        }

    # Extract user data
    core = result.get('core', {})
    user_results = core.get('user_results', {})

    # Extract views
    views = result.get('views', {})

    # Extract quoted tweet
    quoted_status_result = result.get('quoted_status_result', {})
    if quoted_status_result:
        # Recursively normalize quoted tweet
        legacy['quoted_status_result'] = {
            'result': build_tweet_data(quoted_status_result.get('result', {}))
        }

    # Extract retweeted tweet
    retweeted_status_result = legacy.get('retweeted_status_result', {})
    if retweeted_status_result:
        # Recursively normalize retweeted tweet
        legacy['retweeted_status_result'] = {
            'result': build_tweet_data(retweeted_status_result.get('result', {}))
        }

    # Build normalized structure
    normalized = {
        'rest_id': rest_id,
        'legacy': legacy,
        'core': core,
        'views': views,
        'edit_control': result.get('edit_control', {}),
        'note_tweet': result.get('note_tweet', {}),
        'community_results': result.get('community_results', {}),
        'card': result.get('card', {}),
    }

    return normalized
```

---

#### `tweet_from_data()` - Tweet Construction

**Location**: `tweet.py:500-600`

**Implementation**:
```python
def tweet_from_data(client: Client, data: dict) -> Tweet:
    """
    Construct Tweet object from normalized data.

    Args:
        client: Client instance (for method delegation)
        data: Normalized tweet data from build_tweet_data()

    Returns:
        Tweet: Fully constructed Tweet object
    """

    # Normalize data if not already done
    data = build_tweet_data(data)

    # Extract user data and create User object
    user = None
    if 'core' in data and data['core']:
        user_results = data['core'].get('user_results', {})
        if user_results:
            user_data = user_results.get('result', {})
            user_data = build_user_data(user_data)
            user = User(client, user_data)

    # Create Tweet object
    tweet = Tweet(client, data, user)

    # Parse quoted tweet
    if 'quoted_status_result' in data['legacy']:
        quoted_data = data['legacy']['quoted_status_result']['result']
        tweet.quote = tweet_from_data(client, quoted_data)

    # Parse retweeted tweet
    if 'retweeted_status_result' in data['legacy']:
        retweet_data = data['legacy']['retweeted_status_result']['result']
        tweet.retweeted_tweet = tweet_from_data(client, retweet_data)

    return tweet
```

---

### Utility Functions

#### `build_query()` - Advanced Search Builder

**Location**: `utils.py:450-500`

**Implementation**:
```python
def build_query(
    query: str,
    filters: list[tuple[str, str]] | None = None,
    exclude: list[tuple[str, str]] | None = None
) -> str:
    """
    Build Twitter advanced search query string.

    Supports all Twitter search operators:
    - from: (tweets from user)
    - to: (tweets to user)
    - @: (mentions user)
    - #: (contains hashtag)
    - since: (after date)
    - until: (before date)
    - min_retweets: (minimum retweets)
    - min_faves: (minimum likes)
    - min_replies: (minimum replies)
    - filter: (media type: images, videos, links, etc.)
    - lang: (language code)

    Args:
        query: Base search query
        filters: List of (operator, value) tuples to include
        exclude: List of (operator, value) tuples to exclude

    Returns:
        str: Formatted search query

    Examples:
        # Basic search
        build_query('python')
        # Returns: 'python'

        # From specific user
        build_query('python', filters=[('from', 'realpython')])
        # Returns: 'python from:realpython'

        # Multiple filters
        build_query(
            'python',
            filters=[
                ('from', 'realpython'),
                ('min_faves', '10'),
                ('filter', 'images')
            ]
        )
        # Returns: 'python from:realpython min_faves:10 filter:images'

        # Exclude spam
        build_query(
            'python',
            exclude=[('from', 'spam_bot')]
        )
        # Returns: 'python -from:spam_bot'

        # Date range
        build_query(
            'python',
            filters=[
                ('since', '2023-01-01'),
                ('until', '2023-12-31')
            ]
        )
        # Returns: 'python since:2023-01-01 until:2023-12-31'
    """

    # Start with base query
    result = query

    # Add filters
    if filters:
        for operator, value in filters:
            result += f' {operator}:{value}'

    # Add exclusions
    if exclude:
        for operator, value in exclude:
            result += f' -{operator}:{value}'

    return result
```

---

## 6. Data Flow Patterns

### Complete Request Flow Example

**Scenario**: User searches for tweets containing "python"

```python
# User code
tweets = await client.search_tweet('python', 'Latest', count=20)

for tweet in tweets:
    print(tweet.text)
    await tweet.favorite()
```

**Internal Flow**:

```
[1] client.search_tweet('python', 'Latest', 20)
    │
    ├─→ Build GraphQL variables
    │   variables = {
    │       'rawQuery': 'python',
    │       'count': 20,
    │       'product': 'Latest'
    │   }
    │
    ├─→ [2] client.gql._request(Endpoint.SEARCH_TIMELINE, variables, FEATURES)
    │       │
    │       ├─→ Build URL
    │       │   url = 'https://x.com/i/api/graphql/flaR-PUMshxFWZWPNpq4zA/SearchTimeline'
    │       │
    │       ├─→ [3] client.request('POST', url, json={'variables': ..., 'features': ...})
    │       │       │
    │       │       ├─→ Validate client_transaction
    │       │       │   if client.client_transaction is None:
    │       │       │       await client.client_transaction.init(...)
    │       │       │
    │       │       ├─→ Generate transaction ID
    │       │       │   transaction_id = client.client_transaction.generate_transaction_id(
    │       │       │       'POST',
    │       │       │       '/i/api/graphql/flaR-PUMshxFWZWPNpq4zA/SearchTimeline'
    │       │       │   )
    │       │       │
    │       │       ├─→ Build headers
    │       │       │   headers = {
    │       │       │       'Authorization': 'Bearer AAAAAAAAAAAAAAAAAAAAANRILgAA...',
    │       │       │       'X-Csrf-Token': '<csrf_token_from_cookies>',
    │       │       │       'X-Client-Transaction-Id': '<transaction_id>',
    │       │       │       'Content-Type': 'application/json',
    │       │       │       'User-Agent': 'Mozilla/5.0...'
    │       │       │   }
    │       │       │
    │       │       ├─→ [4] httpx.AsyncClient.post(url, headers=..., json=...)
    │       │       │       │
    │       │       │       └─→ HTTP Request to Twitter
    │       │       │           POST https://x.com/i/api/graphql/.../SearchTimeline
    │       │       │           Headers: {...}
    │       │       │           Body: {"variables": {...}, "features": {...}}
    │       │       │
    │       │       ├─→ [5] Twitter API Response
    │       │       │   {
    │       │       │     "data": {
    │       │       │       "search_by_raw_query": {
    │       │       │         "search_timeline": {
    │       │       │           "timeline": {
    │       │       │             "instructions": [
    │       │       │               {
    │       │       │                 "entries": [
    │       │       │                   {
    │       │       │                     "content": {
    │       │       │                       "itemContent": {
    │       │       │                         "tweet_results": {
    │       │       │                           "result": {
    │       │       │                             "rest_id": "1234",
    │       │       │                             "legacy": {...}
    │       │       │                           }
    │       │       │                         }
    │       │       │                       }
    │       │       │                     }
    │       │       │                   }
    │       │       │                 ]
    │       │       │               }
    │       │       │             ]
    │       │       │           }
    │       │       │         }
    │       │       │       }
    │       │       │     }
    │       │       │   }
    │       │       │
    │       │       └─→ Check status code
    │       │           if status_code != 200:
    │       │               raise_exceptions_from_response(status_code, error_code)
    │       │
    │       └─→ Return response.json()
    │
    ├─→ [6] Parse response
    │   │
    │   ├─→ find_dict(response, 'entries', find_one=True)
    │   │   Returns: list of entry dicts
    │   │
    │   ├─→ For each entry:
    │   │   │
    │   │   ├─→ If entry_id starts with 'tweet-':
    │   │   │   │
    │   │   │   ├─→ find_dict(entry, 'result', find_one=True)
    │   │   │   │   Returns: tweet data dict
    │   │   │   │
    │   │   │   ├─→ [7] build_tweet_data(tweet_data)
    │   │   │   │   Normalizes structure
    │   │   │   │
    │   │   │   ├─→ [8] tweet_from_data(client, normalized_data)
    │   │   │   │   │
    │   │   │   │   ├─→ Extract user data
    │   │   │   │   ├─→ build_user_data(user_data)
    │   │   │   │   ├─→ Create User(client, user_data)
    │   │   │   │   └─→ Create Tweet(client, tweet_data, user)
    │   │   │   │
    │   │   │   └─→ Append to tweets list
    │   │   │
    │   │   └─→ If entry_id starts with 'cursor-bottom-':
    │   │       Extract next_cursor
    │   │
    │   └─→ tweets = [Tweet, Tweet, Tweet, ...]
    │       next_cursor = '...'
    │
    └─→ [9] Create Result object
        │
        ├─→ Create fetch_next function
        │   fetch_next = partial(
        │       client.search_tweet,
        │       query='python',
        │       product='Latest',
        │       count=20,
        │       cursor=next_cursor
        │   )()
        │
        └─→ Return Result(
                results=tweets,
                next_cursor=next_cursor,
                fetch_next_result=fetch_next
            )

[10] User iterates over Result
     for tweet in tweets:
         │
         ├─→ Result.__iter__() yields tweets
         │
         └─→ For each tweet:
             print(tweet.text)
                 │
                 └─→ Tweet.text property
                     return self._legacy['full_text']

             await tweet.favorite()
                 │
                 └─→ [11] Tweet.favorite()
                     return await self._client.favorite_tweet(self.id)
                         │
                         └─→ [12] client.favorite_tweet(tweet_id)
                             │
                             ├─→ Build variables
                             │   variables = {'tweet_id': tweet_id}
                             │
                             ├─→ client.gql._request(
                             │       Endpoint.FAVORITE_TWEET,
                             │       variables,
                             │       FEATURES
                             │   )
                             │   [Same flow as steps 3-5]
                             │
                             └─→ Return updated Tweet object
```

---

## 7. Authentication & Security

### Multi-Layer Security Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                   SECURITY LAYER STACK                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  [Layer 1] Session Management                                   │
│  ├─ Cookie-based authentication                                 │
│  ├─ CSRF token (ct0 cookie)                                     │
│  ├─- Auth token (auth_token cookie)                             │
│  └─ Persistent session (cookies saved to file)                  │
│                                                                  │
│  [Layer 2] Bot Detection Evasion                                │
│  ├─ X-Client-Transaction-Id header                              │
│  ├─ Generated from CSS animations + cryptographic keys          │
│  ├─ Changes per request based on endpoint                       │
│  └─ Simulates browser behavior                                  │
│                                                                  │
│  [Layer 3] Account Protection                                   │
│  ├─ Automatic captcha solving (Capsolver integration)           │
│  ├─ Account unlock flow                                         │
│  ├─ 2FA support (TOTP)                                          │
│  └─ SSO authentication                                          │
│                                                                  │
│  [Layer 4] Request Signing                                      │
│  ├─ Bearer token (public, from web app)                         │
│  ├─ Guest tokens (for unauthenticated access)                   │
│  ├─ User-Agent spoofing                                         │
│  └─ UI metrics (browser simulation)                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Authentication Flow State Machine

```
┌─────────────┐
│   INITIAL   │
│   (Guest)   │
└──────┬──────┘
       │
       │ login() called
       │
       ▼
┌─────────────┐
│  GET_GUEST  │  ◄──────────────┐
│   TOKEN     │                 │
└──────┬──────┘                 │
       │                        │
       │ guest_token obtained    │
       │                        │
       ▼                        │
┌─────────────┐                 │
│ INIT_FLOW   │                 │
│             │                 │
└──────┬──────┘                 │
       │                        │
       │ flow_token obtained     │
       │                        │
       ▼                        │
┌─────────────┐                 │
│   ENTER     │                 │
│  USERNAME   │                 │
└──────┬──────┘                 │
       │                        │
       │ username submitted      │
       │                        │
       ├─────────────┐          │
       │             │          │
       ▼             ▼          │
┌─────────────┐ ┌─────────────┐│
│   ENTER     │ │ ENTER_EMAIL ││
│  PASSWORD   │ │  (if needed)││
└──────┬──────┘ └──────┬──────┘│
       │               │       │
       │◄──────────────┘       │
       │                       │
       │ password submitted     │
       │                       │
       ├───────────────────┐   │
       │                   │   │
       ▼                   ▼   │
┌─────────────┐   ┌─────────────┐
│ AUTHENTICATED│   │  2FA_CHECK  │
│   SUCCESS   │   │             │
└──────┬──────┘   └──────┬──────┘
       │                 │
       │                 │ TOTP submitted
       │                 │
       │◄────────────────┘
       │
       ├──────────────────┐
       │                  │
       ▼                  ▼
┌─────────────┐  ┌─────────────┐
│    SAVE     │  │   LOCKED    │
│   COOKIES   │  │   ACCOUNT   │
└──────┬──────┘  └──────┬──────┘
       │                │
       │                │ unlock() called
       │                │
       ▼                ▼
┌─────────────┐  ┌─────────────┐
│   LOGGED    │  │   SOLVE     │
│     IN      │  │  CAPTCHA    │
└─────────────┘  └──────┬──────┘
                        │
                        │ captcha solved
                        │
                        └────────┐
                                 │
                                 └─► Retry login
```

---

## 8. API Interaction Patterns

### Pattern 1: Simple Resource Fetch

**Use Case**: Get a specific resource by ID

**Example**: Get user by screen name

```python
# User code
user = await client.get_user_by_screen_name('example_user')

# Returns User object directly (not paginated)
print(user.name)
print(user.followers_count)
```

**Internal Flow**:
```
client.get_user_by_screen_name('example_user')
    ↓
Build variables: {'screen_name': 'example_user', 'withSafetyModeUserFields': True}
    ↓
gql._request(Endpoint.USER_BY_SCREEN_NAME, variables, FEATURES)
    ↓
POST https://x.com/i/api/graphql/NimuplG1OB7Fd2btCLdBOw/UserByScreenName
    ↓
Response: {"data": {"user": {"result": {...}}}}
    ↓
Extract user data: find_dict(response, 'result', find_one=True)
    ↓
Normalize: build_user_data(user_data)
    ↓
Construct: User(client, normalized_data)
    ↓
Return User object
```

---

### Pattern 2: Paginated Collection

**Use Case**: Get a collection of resources with pagination

**Example**: Search tweets

```python
# User code
tweets = await client.search_tweet('python', 'Latest', count=20)

# First page
for tweet in tweets:
    print(tweet.text)

# Next page
more_tweets = await tweets.next()

# Continue pagination
for tweet in more_tweets:
    print(tweet.text)
```

**Internal Flow**:
```
client.search_tweet('python', 'Latest', 20)
    ↓
Build variables with cursor (if provided)
    ↓
gql._request(Endpoint.SEARCH_TIMELINE, variables, FEATURES)
    ↓
Response: {"data": {"search": {"timeline": {"instructions": [{"entries": [...]}]}}}}
    ↓
Extract entries: find_dict(response, 'entries')
    ↓
Parse each entry:
    - If 'tweet-': Create Tweet object
    - If 'cursor-': Extract next_cursor
    ↓
Create fetch_next function with cursor
    ↓
Return Result(tweets, next_cursor, fetch_next)
```

---

### Pattern 3: Entity Method Delegation

**Use Case**: Perform actions on entities

**Example**: Tweet actions

```python
# User code
tweet = await client.get_tweet_by_id('1234567890')

# Entity methods delegate to client
await tweet.favorite()
await tweet.retweet()
await tweet.reply('Great tweet!')
```

**Internal Flow**:
```
tweet.favorite()
    ↓
return await self._client.favorite_tweet(self.id)
    ↓
client.favorite_tweet(tweet_id)
    ↓
Build variables: {'tweet_id': tweet_id}
    ↓
gql._request(Endpoint.FAVORITE_TWEET, variables, FEATURES)
    ↓
Response: {"data": {"favorite_tweet": "Done"}}
    ↓
Return updated Tweet object
```

**Why This Pattern?**:
- **Encapsulation**: Entity knows its own ID
- **Convenience**: `tweet.favorite()` instead of `client.favorite_tweet(tweet.id)`
- **Consistency**: All actions available from entity
- **Delegation**: Client handles actual API call (authentication, headers, etc.)

---

### Pattern 4: Lazy Loading

**Use Case**: Load expensive data only when accessed

**Example**: Tweet replies

```python
# User code
tweet = await client.get_tweet_by_id('1234567890')

# Replies not loaded yet (tweet.replies is None)

# First access triggers load
replies = await tweet.get_replies()

# Subsequent accesses use cached value
replies_again = await tweet.get_replies()  # Returns cached
```

**Implementation**:
```python
class Tweet:
    def __init__(self, client, data, user):
        self._client = client
        self._data = data
        self.user = user
        self.replies: Result[Tweet] | None = None  # Not loaded yet

    async def get_replies(self, count: int = 40) -> Result[Tweet]:
        """Lazy load replies"""
        if self.replies is None:
            # Load from API
            self.replies = await self._client._get_tweet_replies(
                self.id,
                count
            )
        return self.replies
```

**Benefits**:
- Avoid unnecessary API calls
- Faster initial object creation
- User controls when to load data
- Caching prevents redundant requests

---

## 9. Design Patterns & Best Practices

### 1. Dependency Injection

**All entity classes receive `client` reference**:

```python
class Tweet:
    def __init__(self, client: Client, data: dict, user: User = None):
        self._client = client  # Injected dependency
        # ...

class User:
    def __init__(self, client: Client, data: dict):
        self._client = client  # Injected dependency
        # ...
```

**Benefits**:
- Entities can call client methods
- No global state
- Easier testing (can inject mock client)

---

### 2. TYPE_CHECKING Pattern

**Avoid circular imports while maintaining type hints**:

```python
from __future__ import annotations
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from .client.client import Client
    from .utils import Result

class Tweet:
    def __init__(self, client: Client, data: dict):
        # Type hint works, but import only happens during type checking
        self._client = client
```

**Why**:
- `Client` imports `Tweet`
- `Tweet` needs to reference `Client` for type hints
- `TYPE_CHECKING` is `False` at runtime, so no circular import
- Type checkers see the import and validate types

---

### 3. Property-Based API

**Use `@property` for computed/extracted values**:

```python
class Tweet:
    @property
    def text(self) -> str:
        """Extract text from legacy data"""
        return self._legacy['full_text']

    @property
    def created_at_datetime(self) -> datetime:
        """Convert timestamp to datetime"""
        return timestamp_to_datetime(self.created_at)

    @property
    def media(self) -> list[Media]:
        """Extract and parse media entities"""
        entities = self._legacy.get('extended_entities', {})
        media_list = entities.get('media', [])
        return [_media_from_data(m) for m in media_list]
```

**Benefits**:
- Access like attributes: `tweet.text` instead of `tweet.get_text()`
- Lazy computation (only when accessed)
- Can include complex logic
- Clean, Pythonic API

---

### 4. Generic Pagination

**`Result[T]` provides uniform interface for all paginated resources**:

```python
# All return Result[T]
tweets: Result[Tweet] = await client.search_tweet('python')
users: Result[User] = await client.get_user_followers('1234')
messages: Result[Message] = await client.get_dm_history('5678')

# All have same pagination interface
for item in tweets:
    # ...

next_page = await tweets.next()
```

**Benefits**:
- Consistent API across all resources
- Generic type hints
- Cursor management abstracted away

---

### 5. NamedTuple for Immutable Data

**Simple data structures without methods**:

```python
from typing import NamedTuple

class CommunityCreator(NamedTuple):
    id: str
    screen_name: str
    verified: bool

class VideoVariant(NamedTuple):
    bitrate: int | None
    content_type: str
    url: str
```

**Benefits**:
- Immutable
- Lightweight
- Named fields
- Type hints
- Automatic `__repr__`, `__eq__`, etc.

---

### 6. Error Code Mapping

**Map API error codes to specific exceptions**:

```python
ERROR_CODE_TO_EXCEPTION = {
    37: AccountSuspended,
    64: AccountSuspended,
    187: DuplicateTweet,
    324: InvalidMedia,
    326: AccountLocked,
}

def raise_exceptions_from_response(status_code, error_code):
    if error_code in ERROR_CODE_TO_EXCEPTION:
        raise ERROR_CODE_TO_EXCEPTION[error_code]()
    # ... handle by status code
```

**Benefits**:
- Specific exceptions for specific errors
- Easy to catch specific error types
- Centralized error handling logic

---

### 7. Response Normalization

**Different endpoints return different structures - normalize to standard format**:

```python
def build_tweet_data(tweet: dict) -> dict:
    """
    Normalize variations:
    - 'rest_id' vs 'id_str' vs 'id'
    - Nested 'legacy' vs flat structure
    - Wrapped in 'TweetWithVisibilityResults'

    Always returns:
    {
        'rest_id': str,
        'legacy': dict,
        'core': dict,
        'views': dict
    }
    """
    # ... normalization logic
```

**Benefits**:
- Tweet class only needs to handle one structure
- Easier to maintain
- Robust against API changes

---

## 10. Dependencies & External Integration

### Core Dependencies

```python
# HTTP
httpx[socks]            # Async HTTP client with SOCKS proxy support
                        # Used for all API requests
                        # Supports connection pooling, proxies, async

# HTML Parsing
beautifulsoup4          # Parse HTML for unlock pages, homepage
lxml                    # XML/HTML parser (faster than html.parser)

# Authentication
pyotp                   # TOTP (Time-based One-Time Password) for 2FA
                        # Generates 6-digit codes from secret

# Media
filetype                # Detect media file types from content
                        # Used in upload_media()

# Video Processing
webvtt-py               # Parse WebVTT subtitle files
m3u8                    # Parse HLS video manifests

# JavaScript Execution
Js2Py-3.13              # Execute JavaScript for dynamic data extraction
                        # Used in ClientTransaction for animation parsing
```

### External Service Integration

#### Capsolver (Captcha Solving)

**Service**: https://capsolver.com/

**Purpose**: Solve Arkose captcha challenges for account unlock

**Integration**:
```python
from twikit import Client, Capsolver

# Initialize with Capsolver API key
solver = Capsolver(api_key='CAP-xxxxx')

# Pass to client
client = Client(captcha_solver=solver)

# Automatic unlock on error 326
await client.login(...)  # If account locked, auto-solves and unlocks
```

**API Flow**:
```
Twitter returns 326 error
    ↓
Client fetches unlock page HTML
    ↓
Extract assignment_token (Arkose challenge ID)
    ↓
Send to Capsolver API:
    POST https://api.capsolver.com/createTask
    {
        "clientKey": "CAP-xxxxx",
        "task": {
            "type": "FunCaptchaTaskProxyLess",
            "websiteURL": "https://x.com/",
            "websitePublicKey": "...",
            "data": "<assignment_token>"
        }
    }
    ↓
Capsolver returns task_id
    ↓
Poll for solution:
    POST https://api.capsolver.com/getTaskResult
    {"clientKey": "...", "taskId": "..."}
    ↓
Get verification_string
    ↓
Submit to Twitter unlock endpoint
    ↓
Account unlocked
```

---

### Proxy Support

**Twikit supports SOCKS and HTTP proxies**:

```python
# HTTP proxy
client = Client(proxy='http://username:password@proxy.example.com:8080')

# SOCKS5 proxy
client = Client(proxy='socks5://username:password@proxy.example.com:1080')

# No authentication
client = Client(proxy='http://proxy.example.com:8080')
```

**Implementation**:
```python
class Client:
    def __init__(self, proxy: str | None = None, **kwargs):
        if proxy:
            # Parse proxy URL
            transport = AsyncHTTPTransport(
                proxy=httpx.Proxy(proxy)
            )
            self.http = AsyncClient(transport=transport)
        else:
            self.http = AsyncClient()
```

---

## Summary

This architecture documentation has covered Twikit across four layers of abstraction:

1. **Project-Level**: Overall architecture, layer separation, component interaction
2. **File-Level**: Directory structure, file responsibilities, module organization
3. **Class-Level**: Class hierarchy, relationships, design patterns
4. **Function-Level**: Implementation details, algorithms, data flows

Key architectural highlights:

- **Async-first design** for high performance
- **Client-entity pattern** with dependency injection
- **Generic pagination** via `Result[T]`
- **Response normalization** for API inconsistencies
- **Multi-layer security** (session, bot detection, captcha)
- **Lazy loading** for expensive data
- **Type safety** via TYPE_CHECKING pattern
- **Graceful error handling** with exception hierarchy

The codebase demonstrates professional Python practices:
- Type hints throughout
- Clear separation of concerns
- Reusable abstractions
- Defensive programming
- Comprehensive error handling
- Well-documented code

This architecture enables developers to interact with Twitter/X programmatically without official API keys, while maintaining robustness, security, and ease of use.

---

**End of Architecture Documentation**
