# MilkVault — Design Choices and Trade-offs

## The Problem I Wanted to Solve

Paper notebooks can cause problems at dairy collection centers because each agent keeps their own record. If milk gets spoiled, it becomes difficult to know what happened and who handled the batch. People may have different records, which can lead to arguments.

My main goal was to create one shared record that all agents can see. The system also helps identify the risk of spoilage instead of leaving the decision completely to the agent.

## Why I Used a Shared Database

I used Firestore as a shared online database instead of saving the data only on each agent's phone. This allows all agents to see the same batch information and updates in real time. The timestamps are also created by the server, so they do not depend on the phone's clock.

The main disadvantage is that the app needs an internet connection to add or view batches. This can be a problem in villages where the connection may not always be reliable. Since this was a 48-hour project, I decided not to add offline support. Adding incomplete offline support could also affect the main purpose of having one shared record.

In a future version, I can save entries temporarily on the phone and upload them automatically when the internet connection comes back.

## Why I Did Not Add a Login System

Agents only enter their name once, and the app remembers it on that device. There is no password or account creation. I chose this because the app is meant to be quick and simple to use, similar to how a paper notebook is used during a work shift.

The disadvantage is that a person could enter someone else's name. For this prototype, I accepted this limitation because my main focus was simple daily accountability, not complete identity verification.

A future version could use phone-number verification for each agent.

## Why Farmers Are Selected From a List

Each farmer has a fixed ID, such as F001 to F015. Agents select the farmer from a list instead of typing the name every time.

This helps avoid duplicate names caused by different spellings. For example, "Ramesh Yadav" and "ramesh yadav" should not become two different farmer records.

There is also an "Other" option for a farmer who has not yet been added to the list. This makes sure that a real milk delivery is not blocked. However, that entry will not have a permanent farmer ID until the farmer is added properly.

## Why I Use Temperature Bands

I did not ask agents to enter an exact temperature because many collection agents may not have a properly calibrated thermometer with them. Asking for an exact number could also lead to people guessing a value.

Instead, the app provides five temperature ranges and an option for cases where there is no thermometer. In that situation, ambient temperature is assumed.

This keeps the form simple and allows the agent to continue recording the delivery.

## Why the Spoilage Calculation Is Simple

The app uses a simple lookup table to estimate how much safe time is left for the milk. The estimate is roughly 24 hours for properly chilled milk and can go down to about 1 hour for milk that is delivered very hot.

I chose this simple approach instead of using a complicated formula. Milk spoilage depends on more than temperature, including the starting level of bacteria. Because of this, a complicated formula would not automatically give a more accurate answer.

These values are only planning assumptions and are not a certified food safety standard. Before using the system in real dairy operations, the values should be checked with FSSAI or the relevant local dairy cooperative guidelines.

## Why I Keep History Instead of Replacing It

Whenever something happens to a batch, such as creating it, pouring it, or raising a dispute, the app adds a new event to the history.

The previous information is not quietly replaced. This makes it possible to see what happened, who performed an action, and when it happened.

This is an important part of the project because it helps when there is a disagreement about which batch caused a problem.

## Why the Vat Has a Separate Monitor

An individual batch and the main milk tank are not the same thing. A batch can still be safe even when the tank is almost full or when the tank is waiting too long for pickup.

For this reason, the app shows the vat status separately. It includes the current tank level compared with its capacity and a countdown showing when the pooled milk becomes overdue for collection.

The countdown starts when the first milk is poured into an empty vat. It only resets when an agent confirms that the tanker has arrived. I chose this instead of resetting it every day because tanker collection times do not always follow a fixed daily schedule.

## Why the SMS Feature Does Not Send an Actual SMS

The project requirements do not allow paid SMS APIs, so the app does not send real SMS messages automatically.

Instead, the "Generate pickup summary" feature creates a simple text message containing information such as pending milk volume, vat level, and urgent batches. The agent can then use their phone's messaging option or copy the message and send it to the driver.

The agent still chooses the driver's number and decides when to send the message.

## Edge Cases I Handle

* **No thermometer:** The agent can choose the available temperature fallback instead of being blocked.
* **Pouring a flagged batch:** The agent must enter a reason before doing so. The reason stays with the batch record.
* **Batch disagreement:** A dispute is added as a new event without changing the original record.
* **Two agents working at the same time:** Each batch is stored separately, so entries do not overwrite each other.
* **Phone restart:** The data is stored in Firestore, so submitted entries are not lost if the phone restarts. Only an entry that was not submitted may be lost.
* **Old countdown on an open page:** The remaining time is calculated again whenever the page is shown, so it does not depend on how long the page has been left open.

## Main Trust Features

1. All agents can see the same batch information.
2. The server provides the timestamps instead of relying on the phone's clock.
3. New actions are added to the history instead of replacing old information.
4. The app calculates the risk using the recorded time and temperature.
5. If an agent overrides a risk warning, they must provide a reason.

## What I Would Improve Next

If I continue developing MilkVault, I would focus on the following improvements:

* Improve the Firestore security rules so that the history cannot be changed even if someone tries to bypass the app.
* Add offline support for areas with poor internet connectivity.
* Add simple phone-number verification for agents.
* Test and improve the spoilage estimates using real data from local dairy cooperatives.
