# Project 1. A Simple Notification System

**Chapters used:** 10 (functions), 12 (dictionaries, sets), 13 (error
handling), 14 (OOP).

## Description

Write a small program that sends notifications to users through
different channels: **email**, **SMS** and **push**. The program does not
send real messages — "sending" means printing the message to the screen
and saving it to the history.

## Requirements

1. **The `User` class** — a name, an email, a phone number and a set of
   preferred channels (for example, `{"email", "push"}`).
   - Validate the data in `__init__`: the name is not empty, the email
     contains `@`, the phone number contains only digits (and may start
     with `+`). Raise `ValueError` for invalid values.

2. **The base class `Notification`** — a recipient (`User`) and a text.
   - The method `send()` must be overridden in subclasses
     (in the base class it raises `NotImplementedError`).
   - The text must not be empty.

3. **Three subclasses** — `EmailNotification`, `SMSNotification`,
   `PushNotification`. Each one overrides `send()` and prints the message
   in its own format, for example:
   ```
   [EMAIL] to anna@example.com: Your order has been shipped
   [SMS]   to +380501234567: Your order has been shipped
   [PUSH]  to Anna: Your order has been shipped
   ```
   - An SMS is limited to **160 characters**: a longer text is cut and
     ends with `...`.

4. **The `NotificationService` class** — the main part of the system:
   - `add_user(user)` — register a user (the same email cannot be added
     twice);
   - `notify(user, text)` — send the message through **every** preferred
     channel of the user;
   - `broadcast(text)` — send the message to all registered users;
   - `history()` — a list of all sent notifications (channel, recipient,
     text);
   - `stats()` — a dictionary with the number of sent messages per
     channel, for example `{"email": 3, "sms": 1, "push": 2}`.

5. If sending through one channel fails (for example, the user has no
   phone number), the error must be **caught and printed**, and the other
   channels must still work.

6. Every class and method has a docstring; `__str__` gives readable output.

7. Demonstration in an `if __name__ == "__main__":` block: create 3 users
   with different channels, send one personal notification and one
   broadcast, then print the history and the statistics. Add at least
   **3 checks via `assert`**.

## Extra Tasks (Optional)

- Save the history to a file `notifications.txt` and load it back
  (Chapter 13).
- Add a **priority** (`low`, `normal`, `high`) and print the history
  sorted by priority (Chapter 11).
- Let a user turn a channel on or off: `user.enable("sms")`,
  `user.disable("email")`.

## Grading

| Grade | Requirements |
|-------|--------------|
| 5 | all requirements, readable code, errors handled |
| 4 | all classes work, small flaws in validation or formatting |
| 3 | the classes and `notify()` work, but no `stats()` or error handling |
| 2 | only some classes are written |
