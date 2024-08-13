# exponent-server-sdk-python-async

This repo is made for the async version of the [exponent-server-sdk-python](https://github.com/expo-community/expo-server-sdk-python) made by [Bahadır Araz](https://github.com/bahadiraraz).

## Installation

```
pip install exponent-server-sdk-async
```

## Usage
Use to send push notifications to Exponent Experiences from a Python server.
[Full documentation](https://docs.expo.dev/push-notifications/sending-notifications/#http2-api) on the API is available if you want to dive into the details.
```python
<<<<<<< HEAD
from exponent_server_sdk import (
    DeviceNotRegisteredError,
    PushClient,
    PushMessage,
    PushServerError,
    PushTicketError,
)
import os
import requests
from requests.exceptions import ConnectionError, HTTPError

# Optionally providing an access token within a session if you have enabled push security
session = requests.Session()
session.headers.update(
    {
        "Authorization": f"Bearer {os.getenv('EXPO_TOKEN')}",
        "accept": "application/json",
        "accept-encoding": "gzip, deflate",
        "content-type": "application/json",
    }
)

# Basic arguments. You should extend this function with the push features you
# want to use, or simply pass in a `PushMessage` object.
def send_push_message(token, message, extra=None):
    try:
        response = PushClient(session=session).publish(
            PushMessage(to=token,
                        body=message,
                        data=extra))
    except PushServerError as exc:
        # Encountered some likely formatting/validation error.
        rollbar.report_exc_info(
            extra_data={
                'token': token,
                'message': message,
                'extra': extra,
                'errors': exc.errors,
                'response_data': exc.response_data,
            })
        raise
    except (ConnectionError, HTTPError) as exc:
        # Encountered some Connection or HTTP error - retry a few times in
        # case it is transient.
        rollbar.report_exc_info(
            extra_data={'token': token, 'message': message, 'extra': extra})
        raise self.retry(exc=exc)

    try:
        # We got a response back, but we don't know whether it's an error yet.
        # This call raises errors so we can handle them with normal exception
        # flows.
        response.validate_response()
    except DeviceNotRegisteredError:
        # Mark the push token as inactive
        from notifications.models import PushToken
        PushToken.objects.filter(token=token).update(active=False)
    except PushTicketError as exc:
        # Encountered some other per-notification error.
        rollbar.report_exc_info(
            extra_data={
                'token': token,
                'message': message,
                'extra': extra,
                'push_response': exc.push_response._asdict(),
            })
        raise self.retry(exc=exc)
=======
import asyncio
import os
import httpx
from loguru import logger
from dotenv import load_dotenv
from exponent_server_sdk_async import (
    AsyncPushClient,
    PushMessage,
    DeviceNotRegisteredError,
    PushTicketError,
    PushServerError
)

# Logger setup
logger.add("debug.log", rotation="1 week", compression="zip")
load_dotenv()
logger.debug("Using Expo Token: {}", os.getenv('EXPO_TOKEN'))


async def send_push_message_to_multiple(tokens, message, extra, title, navigate_to=None):
    async with httpx.AsyncClient(headers={
        "Authorization": f"Bearer {os.getenv('EXPO_TOKEN')}",
        "Accept": "application/json",
        "Content-Type": "application/json",
    }) as session:
        push_client = AsyncPushClient(session=session)
        push_messages = [
            PushMessage(
                to=token,
                body=message,
                data={'extra': extra, 'navigateTo': navigate_to},
                badge=1,
                title=title,
                sound="default"
            ) for token in tokens
        ]
        try:
            push_tickets = await push_client.publish_multiple(push_messages)
            for push_ticket in push_tickets:
                if push_ticket.is_success():
                    logger.info("Notification sent successfully to: {}", push_ticket.push_message.to)
                else:
                    logger.warning("Failed to send notification to: {}, Error: {}", push_ticket.push_message.to, push_ticket.message)
        except PushServerError as exc:
            logger.error("Push server error: {}", exc)
            raise
        except DeviceNotRegisteredError as exc:
            logger.error("Device not registered error: {}", exc)
            raise
        except PushTicketError as exc:
            logger.error("Push ticket error: {}", exc)
            raise


async def main():
    tokens = [
        "ExponentPushToken[xxxxxxxxxxxxxxxxxxxxxx]",
        "ExponentPushToken[yyyyyyyyyyyyyyyyyyyyyy]",
        "ExponentPushToken[zzzzzzzzzzzzzzzzzzzzzz]"
    ]
    extra_parameters = {"key": "value"}

    # Send a simple push message to multiple recipients
    await send_push_message_to_multiple(tokens, "Hello Multi-User", extra_parameters, "Greeting")

    # Navigate to home data for multiple recipients
    await send_push_message_to_multiple(tokens, "Navigate to home", extra_parameters, "Home", "/(tabs)/(home)")


if __name__ == "__main__":
    asyncio.run(main())
>>>>>>> 91a8257 (2.1.6 version)
```
