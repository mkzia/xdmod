---
title: User Dashboard (Beta)
---

Known Issues
------------

* Edit In Metric Explorer for Dashboard Charts

The "Edit In Metric Explorer" button on a dashboard chart will not load the
dashboard chart if there already exists a chart with an identical name saved in
the metric explorer.  The workaround for this is to change the name of
 the existing saved chart in the metric explorer so that it does not
match the dashboard chart name.

Developer Guide
---------------

New components can be added to the dashboard as follows:

1. Create a javascript class that inherits from `Ext.ui.Portlet`. Conventionally this should be placed in `html/gui/js/modules/dashboard`
1. Edit the `etc/assets.json` file to include the new javascript artifact (and any other artifacts such as css files).
1. Edit the `etc/roles.d/summary.json` file to add the portlet information for the roles that should see it.


An example component javascript class is shown below.

```javascript

/**
 * XDMoD.Module.Dashboard.DeveloperComponent
 *
 * This is an example panel that can be used as a template.
 */

Ext.namespace('XDMoD.Module.Dashboard');

XDMoD.Module.Dashboard.DeveloperComponent = Ext.extend(Ext.ux.Portlet, {

    layout: 'fit',
    autoScroll: true,
    title: 'XDMoD software developer info',
    bbar: {
        items: [{
            text: 'Reset Layout',
            handler: function () {
                Ext.Ajax.request({
                    url: XDMoD.REST.url + '/summary/layout',
                    method: 'DELETE'
                });
            }
        }]
    },

    /**
     *
     */
    initComponent: function () {
        var aspectRatio = 0.8;
        this.height = this.width * aspectRatio;

        this.items = [{
            itemId: 'console',
            html: '<pre>' + JSON.stringify(this.config, null, 4) + '</pre>'
        }];

        XDMoD.Module.Dashboard.DeveloperComponent.superclass.initComponent.apply(this, arguments);
    },

    listeners: {
        /**
         * duration_change event gets fired when the duration settings
         * in the top toolbar are changed by the user or when the refresh
         * button is clicked.
         *
         * A typical portlet will reload its content with the updated
         * duration parameters.
         */
        duration_change: function (timeframe) {
            var console = this.getComponent('console');
            console.body.insertHtml('beforeBegin', '<pre>' + JSON.stringify(timeframe, null, 4) + '</pre>');
        },

        /**
         * the portlet should refresh itself if the refresh event fires
         */
        refresh: function () {
            var console = this.getComponent('console');
            console.body.insertHtml('beforeBegin', '<pre>refresh event fired</pre>');
        }
    }
});

/**
 * The Ext.reg call is used to register an xtype for this class so it
 * can be dynamically instantiated
 */
Ext.reg('xdmod-dash-developer-cmp', XDMoD.Modules.Dashboard.DeveloperComponent);
```

In this example the javascript class would be put in the file `html/gui/js/modules/dashboard/DeveloperComponent.js`
The `etc/assets.json` would then be updated to add the path to the javascript file
to the `xdmod.portal.js` array.

```json
{
    "xdmod": {
        "portal": {
            "js": [
                "gui/js/modules/dashboard/DeveloperComponent.js"
                ...
            ]
        }
    }
}
```

Similarly the `roles.d/summary.json` file would be edited to add the portlet to the roles that
will see it. In the example below the portlet is added to the summary page for
the center director role (`cd`). The `type` setting should match the xtype string
in the `Ext.reg` call in the javascript.  The `name` setting must be set to a
unique value. It is recommended to set this to a human understandable
description of the portlet. The `config` setting  may contain any valid json
and is available to the portlet as the `this.config` property.

```json
{
    "roles": {
        "cd": {
            "summary_portlets": [{
                "name": "Example Developer Component",
                "type": "xdmod-dash-developer-cmp",
                "location": {
                    "row": 2,
                    "column": 1
                },
                "config": {
                    "property1": "any valid json",
                    "property2": 3
                }
            }]
        }
    }
}
```

The default position of the portlet is specified in the `location` property.
The `location` setting has two mandatory properties the zero indexed row (`row`)
and zero indexed column (`column`). The row value should be less than 1024.
The summary page does not support gaps between the portlets so the row number
specifies the relative order of the portlets not their absolute position.
If multiple portlets specify the same location then they will be placed
in the specified column and their relative row position will be in alphabetical
order by the `name` property. All portlets must have unique names.
