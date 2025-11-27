I need a system to delete or hide listings. If I delete a listing I don't want to physically remove it, I want it deleted using a status, in that way we will be able to know if records in imports should be imported, updated, or ignored (if a record is 'deleted' then don't import)

We already have listings.listing_visible and listings.listing_status available to control visibility and status.

So the backend should be able to view or filter deleted or hidden listings.

The front end should not show hidden or deleted listings, but if a trip contains a hidden or deleted listing, the trip should still display the listing with a visible indicator that it is a deleted or hidden listing.

DESIGN

