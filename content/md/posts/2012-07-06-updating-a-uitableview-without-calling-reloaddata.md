{:layout :post, :title "\"Updating a UITableView without calling reloadData\"", :date "2012-07-06 11:21"}
For a new app I've been working on for the past few days I wanted to a nice "inline" way of gathering data from the user. Typically, if I've needed to have a user put in more than a single line of text, I would have popped the user to a different screen that has a UITextView as the sole point of focus, get them to type in the data, and then have them navigate back and have the data appear within a tableview.

For this new app, I wanted them to be able to enter large amounts of text inline within a cell of a tableview. This also means that the cell would have to dynamically grow and shrink, as the user is entering the data. The UITextView (where the user is entering the data) would need to resize itself based on the users input, as well as the UITableViewCell that contains the text view, and have the UITableView adjust on the fly to the users input.

What I needed to happen was for the UITableView to go through the process of querying the height of each of the cells via the UITableViewDelegate's tableView:heightForRowAtIndexPath: method, and executing the necessary layout code to push/pull the cells around as an individual cell expands or shrinks. The other complication is that the UITextView that the user is typing in cannot lose focus (i.e., it cannot resignFirstResponder). The obvious way to get the UITableView to perform layout is to call [tableview reloadData], however, this causes the UITextView to lose focus, and the keyboard disappears. This seems to happen when creating the cells via the tableview:cellForRow:atIndexPath: method.

Then I found a sneaky trick.

If you execute the following code the table view will query the height for the individual cells, and lay them out, but not reload the cells, and not cause the UITextView to resignFirstResponder.

    [tableview beginUpdates];
    [tableview endUpdates];

So, now I have cells that can grow and shrink dynamically, and not lose focus for the user as they are inputting data.
