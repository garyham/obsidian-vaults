# Microservices

The backend is composed of the following microservices.

| service     | role                                                         |
| ----------- | ------------------------------------------------------------ |
| Visit       | A vendor visits a location, on a date, for a duration.  A customer may search an area within a time window to see which vendors will be there.  A visit is further broken down into collection slots and each collection slot has a maximum capacity.  In this way demand can be evened out.    A unique set of promotions can also be set for each visit. |
| Vendor      | A vendor may register but certain checks (KYC, identity, permits etc) must be performed before a vendor account is 'live'.   Prior to going live, a vendor may create menus, create promotions, and plan visits but they will not show up in visit searches. |
| Menu        | A vendor may create one or more menus.  Each menu consists of items organised in a layout.  Each item has a pricing structure.  Each item may allow customization.   Menus are used to determine the pricing.  A menu must be associated with a visit but each visit can have a unique menu (in this way items and pricing can be tailored to each visit if necessary). |
| Customer    | A customer may register with the service but are not required to do so to place an order.   Registering allows the customer to save their payment preferences, view current and past orders, and save their favourite vendors/items.  In the future a loyalty feature may be supported for customers. |
| Order       | A customer (or guest) may place an order for a number of items from multiple vendors.  One payment is taken even if there are multiple vendors on the order.  A vendor may see the orders placed with them.   Only registered customers may cancel an order, in part or in whole, and only if fulfilment has not already begun (or is assumed to have begun) |
| Fulfil      | A vendor manage the fulfilment workflow of each order from in-preparation to ready-for-collection.  This workflow may need to be tailored. |
| Review (v+) | A customer may review visit which is associated with a vendor. |
| Promotions  | A vendor (or Yeatery) may create one or more promotions and assign them to a visit.  Promotions are used to calculate a discount for each vendor. |
|             |                                                              |

## Vendor

This is the vendor sign up process.  There will be some checks - both KYC related and for operating licences - but the result will be an entry in vendor store.

Each vendor has a unique identifier.

### Data

| field       | type   | notes                         |
| ----------- | ------ | ----------------------------- |
| idVendor    | guid   |                               |
| name        | string | Pat's Pizza                   |
| logo        | string | url('./pats/pizza.png')       |
| description | string | Perfect Pizza, Popular Prices |

### Data (Sensitive)

| field          | type       | notes |
| -------------- | ---------- | ----- |
| idVendor       | guid (ref) |       |
| paymentAccount |            |       |
| contactName    |            |       |
| contacts       | JSONB      |       |



### Commands



### Events



## Menu

Vendors can have one or more menus.  A vendor may create a new Menu to allow them to assign different pricing or different selection of items.

### Data

Although shown below as tables, we may use JSON or whatever storage mechanism is appropriate.  When they translate to an order, it should look like 

```typescript
interface IOrderItem {
    idOrderItem: guid
    idVisit: guid // the visit (by the vendor) that this menu item is associated with.
    idMenuItem: guid  // a unique item on the menu
    variant: string  // could be size - sm,md,lg, could be course - starter,main. Factor which affects the price. 
    ingredients: string[] // instructions which also affect the price.
    qty: number
}
```




#### Menu
| field  | type   | notes |
| ------ | ------ | ----- |
| idMenu | guid   |       |
| title  | string |       |

#### MenuItem

| field       | type                          | notes                                             |
| ----------- | ----------------------------- | ------------------------------------------------- |
| idMenuItem  | guid                          |                                                   |
| title       | string                        |                                                   |
| description | string                        |                                                   |
| category    | string                        | Allows the UI to group the items (starter, mains) |
| variants    | {label: string, price:number} |                                                   |
| idMenu      | ref to guid                   |                                                   |



### Commands

| command                | parameters                                  | notes                                                        |
| ---------------------- | ------------------------------------------- | :----------------------------------------------------------- |
| createMenu             | {title}                                     | Something that allows you to quickly identify a menu when assigning it to a visit for example, "lunchtime menu, low season". |
| deleteMenu             | {idMenu}                                    | Delete the menu provided it is not in use.                   |
| findMenu               | {text}                                      | searches the title for the text.                             |
| addItemToMenu          | {title,description, variants:{title,price}} |                                                              |
| removeItemFromMenu     | {idItem}                                    |                                                              |
| associateMenuWithVisit | {idVisit,idMenu}                            | Allows a particular menu to be associated with a visit.  If a visit has orders, then we should reject this command. |



### Events Published

none - only because I think no one cares.

### Events Subscribed

evAddVisit - creates an entry such that the associate command can be validated.  If no visit exists then clearly it is an invalid association.

evDeleteVisit - removes the association with a menu.

## Visits

A vendor visits a location, on a date, at a time, and for a duration.  A unique menu can be associated with a visit (in this way items and pricing can be tailored to each visit if necessary).  A unique set of promotions can also be set for each visit.   To manage workload, we break each visit down into a number of timeslots, each of which has a maximum capacity (a dish on the menu can take up 0 or more capacity units depending on how complex it is to fulfil)

### Data

#### Visits

Visits are the key linking data-structure in the system.  When a visit is created the promotions and menu services listen.  A promotion and menu can then be associated with the visit.

| field     | type        | notes                                                        |
| --------- | ----------- | ------------------------------------------------------------ |
| idVisit   | guid        |                                                              |
| arrival   | datetime    | planned arrival time (using datetime as we may span days around midnight) |
| departure | datetime    | planned departure time                                       |
| location  | idLocation  |                                                              |
| recur?    |             | more complex that it might appear.                           |
| idVendor  | ref to guid | which vendor is this visit associated with.                  |
|           |             |                                                              |

### Commands

| command     | parameters                            | notes                                                        |
| ----------- | ------------------------------------- | :----------------------------------------------------------- |
| createVisit | {arrival,departure,location,idVendor} | The idVendor comes from the current session.                 |
| deleteVisit | {idVisit}                             | only possible if visit has no associated orders.  This begs the question of how the Visits service knows if a particular visit has orders.  The answer is that it subscribes to the evOrderPlaced event and keeps track. |
| findVisits  | {arrival,departure,location}          | List of Visits                                               |
|             |                                       |                                                              |
|             |                                       |                                                              |
|             |                                       |                                                              |

### Events

| command        | parameters                                     | notes |
| -------------- | ---------------------------------------------- | :---- |
| evVisitCreated | {idVisit, arrival,departure,location,idVendor} |       |
| evVisitDeleted | {idVisit}                                      |       |
| findVisits     | {arrival,departure,location}                   |       |
|                |                                                |       |
|                |                                                |       |
|                |                                                |       |

## Baskets

### Requirements

1. A user shall be able to add and remove items, from the menu of one or more vendors, to their basket.
2. A user shall be able to tailor an item as allowed by the menu (pick a size, select or deselect ingredients). 
3. A user shall be able to edit an item in the basket.
4. A user shall be able to change the quantity of an item in the basket without leaving the basket.
5. A user shall initiate the purchase with a [[ check out ]] button in the basket.
6. The price in the basket will update to reflect the items.
7. The discounts in the basket will update to reflect the items.
8. A user may perform these actions whether logged in or not.
9. Transitioning from a guest to an authenticated user (log in) shall leave the basket unaffected.  
10. Transitioning from an authenticated user to a guest (log out) shall empty the basket.
11. There is no requirement to synchronise baskets between devices.  If the same user is logged in on two different devices, each basket is independent.
12. A basket shall contain no sensitive information.
13. A basket may be held in client storage.
14. A basket shall empty after a defined period of inactivity even if logged in.



A Basket is an embryonic Order.  

When the client side initialises, it has some form of memory (local/session storage or cookie) of the current Order and it uses this to retrieve the current Order.  If their is no Order, then the Basket is empty.

When a user adds an item to the Basket, we need to create a new Order and add and Item to it.  The Basket will persist over reloads.  Consider what happens if we use a session cookie.  If the user closes the browser, then the cookie vanishes which means there is no Basket on the client and the server is left with a dangling order (which can be cleaned up in due course)

POST /basket {initial item} < setCookie(BASKET, idOrder).

POST /basket/items {item} < idItem

DEL /basket/items/:idItem

DEL /basket

POST /basket/checkout

GET /basket



What happens if a user can customise an item in the menu?  This process begins with them clicking [[ customise ]] beside a menu item.  This will create an item which could go in the basket.  We could assign it an guid, allow the user to edit it and then add it to the basket when they [[ save to basket ]] or delete it if the click [[ cancel ]].  If the item is in the basket, they see the same edit screen.  How do we navigate?  Looking at Domino's its /pizza/menu?id=xxx and then /basket/1.

A Basket contains a list of intended visits and items for each visit.  A basket is not an order.  A basket contains no sensitive information.  A basket is maintained on local storage.  When a user adds an item to a basket, it is updated in the local storage and then we call a service on the server to determine what discounts to apply.

Each device used establishes an independent basket.  The precludes the ability to edit a basket across multiple devices which is not envisaged as a likely use-case. 

A user may add to the basket anonymously or as an authenticated user.  Their is no correlation between a user and a basket.

The basket will only persist for a short period.  The user session may persist considerably longer.

This should provoke a client-side event which will force various client-side components to request information from the server.  One such entity is the BasketUI.  This will request an OrderSummary.   If the user had not put anything in the BasketUI prior to logging in, then the server simply returns an empty summary.  If the user had put something in the Basket (and therefore there is a cookie) then we use it to determine the summary.

If the user had nothing in the BasketUI (and therefore no cookie), then we should check to see if the user has any existing Orders with Items.  If so, we set the cookie to refer to this Order and return the summary associated with this Order.

A Basket, while seemingly simple, has a ton of functionality and state beyond the obvious items.    It keeps track of what Items the customer wants to purchase.  It is also the resource which orchestrates the placement of an order (reserve timeslots -> take payment -> place order) including error paths.  It also needs to interact with Promotions (which calculates discounts) and Menus (to get the pricing).  There is also the possibility of a Basket being for a guest or a logged-in user and the possible transaction from guest to logged-in user (where both may have an open basket).  We also have to consider a single user having multiple connections to Yeatery with a mix of guest and logged in sessions.  Does the basket have state which reflects whether a user is in the process of checking out (and at what stage in the process) with this basket...I think it does.  It may event need to hold information to allow actions to be undone although this feels like it should be distributed to the relevant microservice.

What is clear is than once logged in, their is a single basket shared between all sessions where the same person is logged in.

The same person may also visit the site without logging in and have multiple guest baskets. 

??? What does it mean to add an item to a basket.  ???  It has to be only the item, variant and ingredients.  The pricing needs to come from the server otherwise we are open to attack.  It's the same with discount calculation.  Domino's seems to do it by asking for the pricing for the Basket each time an item is added, removed or changed.  The basket is stored on the client. 

??? Is a Basket is created and destroyed or are they persistent ???

??? If created and destroyed, then when ???  

* We could create a basket when the user visits the site if no valid basket exists.  The idBasket could be stored in a cookie.   If a user subsequently logs in, we transfer the contents of this basket to their personal one.  **I have no idea where this functionality should be implement, nor how we trigger it.  The act of logging will be managed by a different microservice, which could emit an event with the guest idBasket of the guest, and then deletes from the session (remove cookie).  The Basket service could merge the users basket and the guest basket before deleting the guest basket.  This all feels monolithy rather then event-sourcey.  The fundamental tenet of ES is that the state is the summation of all events.  If I replay these events the guest basket could not be acted on.  I'm also reliant on cookies in the UI to work.**  I did consider allowing both to persist and the Basket UI to be the aggregate of both but this would not reflect into other sessions.`
* We could create a basket when the user adds their first item (and destroy it when they remove their last item).   This feels overly complex but may be necessary to avoid the unnecessary creation of a basket before log in.

OK, lets try and simplify the problem.  A Basket is a means of tracking what the customer wants to order.  It contains a number of {idVisit, idItem, quantity} records. Customers add and remove items regularly.  They can also change the quantity.

There are two Basket UI views.  A BasketSummary (price after discount, and items), and a BasketDetailsUI (visits with items, pricing and discounts).

The Basket UI needs to be populated with the vendor information (keyed by the visit), pricing information (keyed by the idVisit, idItem), and the promotion information (keyed by the idVisit, idItem).  

To get this information the client needs to query the server and build a composite view.  The vendor information comes from the vendor associated with the Visit. The pricing information comes from the current menu associated with the Visit.  The discount information could change each time an item is added or removed so it needs to be asked for each time. 

The Server may also remove items from the Basket if the visit is no longer in scope (passed the latest slot time).  The Basket UI needs to be updated to reflect this change.

Their is no need to be logged in to create a Basket but should a guest add items to a basket and then log in, the contents of the guest basket should be merged into their permanent basket.

#### Placing an order (checkout)

The act of placing an order follows a flow - reserve timeslots > make payment > place order.  The items remain in the Basket until the order is successfully placed.  The flow may fail at any point in the flow so actions taken may need to be reversed in some manner (by sending the anti-event?).   We need to guard against timeslots being held for 'zombie' orders.

What entity orchestrates this flow?  Is it the client?  Is it a process on the server using HATEOS to direct the client?  It has to be the client which orchestrates this flow.  From the servers perspective any flow state must be in the request (query string) and it acts upon that.  The flow state can grow with each step and should be tamperproof. 

It all starts with a [[ checkout ]] button on the Basket UI.  Clicking this could navigate us to a 'timeslot' screen which loads timeslots for each visit in the basket and their remaining capacity.  The capacity required for a visit is determined by the number of items in the basket (do all items contribute equally to the capacity e.g. making a burger vs picking up a drinks bottle)  A timeslot must be selected for each visit to progress.  This timeslot must be able to cope with all the capacity of the order (it may be possible for an order to span 2 timeslots to fulfil the order)  It should also be possible to change these timeslots again before completing the order.  When a timeslot is selected to address the visit, the capacity is reserved on the server.

Having reserved timeslots we move to payment.  Payment is taken by some means, as yet to be determined, but could fail permanently.  Payment tends to be a 2 step process.  Step 1 we take the payment details and amount and get a token.  Step 2 is that the user confirms that they want to place the order and we use the token to actually take the order.

Once the payment phase is complete, we place the order and empty the basket.  We may also need to confirm timeslots.

The person may also just not finish placing the order or just take to long.

In all the failure e need to release the reserved capacity somehow. 

How do we allocate timeslots in the ordering flow.

### Vendor

How does a vendor see the world?

From a vendors perspective they need to see a pipeline orders by timeslot for each visit.  Each order has a list of items which need to be ready to be collected at a specific time.

Orders need to be picked one by one and fulfilled.  Many orders could be being fulfilled at the same time.  Each order follows a process flow to fulfilment which may need to be tailored to the business.  A starter flow could be in-prep > prepared > making >  fulfilled.  How the vendor organises their operations to manage this process not our concern. 



