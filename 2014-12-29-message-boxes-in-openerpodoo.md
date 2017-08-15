# Message Boxes in OpenERP


_Edit: This technique does not appear to work in Odoo/OpenERP 8_

Being able to show a user a message is a pretty basic, important piece of
functionality. It took me a while to figure out how to trigger a user-visible
message from the server in OpenERP, but I eventually managed it. Given
[the](http://stackoverflow.com/questions/17017215/warning-message-not-working-
on-openerp) [answers](http://stackoverflow.com/questions/15654286/how-can-i
-display-log-message-in-openerp-web-client) on Stack Overflow, I figured I
should share this valuable finding with the rest of the world.

To set the stage, I was writing code to cancel postage on packages at the
click of a button in the package tree view. Sometimes the postage cannot be
cancelled for one reason or another and the shipping server returns a message
explaining why. In that case I wanted to display whatever message the shipping
server was returning to the end user.

OpenERP 7 comes with two client actions that will result in a
[Growl](http://growl.info/)-like message in the user's browser: "action_warn"
and "action_info." They both take three parameters: "title," "text," and
"sticky." The first two are the title text and the message text, respectively,
as strings. The third is a boolean indicating whether the message should
remain on the screen until the user closes it by clicking the "x" in the
corner of it. Setting this to false will result in the message fading away on
its own after a few seconds.

To trigger these client actions from a server action, just have the server
action return a dictionary like this one:

    {  
        'type': 'ir.actions.client',  
        'tag': 'action_warn',  
        'name': 'Failure',  
        'params': {  
           'title': 'Postage Cancellation Failed',  
           'text': 'Shipment is outside the void period.',  
           'sticky': True  
        }  
    }
  
Here's a fuller example for those of you who need more illustration:

    from openerp.osv import fields, osv  
    from openerp.tools.translate import _


    class growling_growler(osv.osv):  
        _name = "growling.growler"  
        _columns = {  
            'name': fields.char(  
                'Name', size=32, required=True  
            )  
        }


        def growl(self, cr, uid, ids, context=None):  
            return {  
                'type': 'ir.actions.client',  
                'tag': 'action_info',  
                'name': _('Growl'),  
                'params': {  
                    'title': _('Grr...'),  
                    'text': _('I am growling at you!'),  
                    'sticky': False  
                }  
            }


And the XML:

    <?xml version="1.0" encoding="UTF-8"?>
    <openerp>
        <data>
            <record id="growling_growler_tree"
                    model="ir.ui.view">
                <field name="name">
                    growling.growler.tree
                </field>
                <field name="model">
                    growling.growler
                </field>
                <field name="arch" type="xml">
    <tree string='Growling Growlers'>
                        <field name='name'&#47;>
                        <button string="Growl!"
                                name="growl"
                                type="object" />
                    </tree>
                </field>
            </record>
        </data>
    </openerp>
