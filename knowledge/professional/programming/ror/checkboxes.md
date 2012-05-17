---
layout: master
title: Rails View checkboxes
---

## Overview

Delete Multiple selected items.

Using checkbox

## Views

views\xxx\index.html.erb

    <% form_tag deletes_xxx_path, :method => :put do %>
      <ul>
      <% for x in @xxx %>
       ( or  <% @xxx.each do |x| %> )
        <li>
          <%= check_box_tag **"xxx_ids[]"**, x.id %>
          <%= task.name %>
        </li>
      <% end %>
      </ul>
      <%= submit_tag "Delete selected" %>
    <% end %>
    
**Add confirmation**

    <%= submit_tag "Delete selected", :confirm => 'Look before you leap.\n Are you sure?'%>

## Controller

controllers\xxx_controller.rb

      def deletes
        logger.info "deletes: #{params[:xxx_ids]}"
    
        _ids = params[:xxx_ids]

        if _ids.nil?
                redirect_to info_logs_url, :notice => "No items selected."
                return
        end

        @xxx = Xxxx.find(_ids)
        @xxx.each do |log|
            logger.info "delete item: #{log.id}"
            log.destroy
    
            ## Delete the logs
            dir_path = "public/newlogs/#{log.id}"
            logger.info "delete dir: #{dir_path}"
            system("(rm -rf #{dir_path})")
        end
    
       redirect_to xxxs_url, :notice =>  "Successfully destroy selected logs}"
      end


## routes

     map.resources :xxx, :collection => { :deletes => :put }

## Checkall and uncheckall

Using getElementsByClassName

###1 Add check_box in a class 

example: x-select

        <td><%= check_box_tag("xxx_ids[]", xx.id, false, :class=>'x-select') %></td>

###2 Add javascript fuction

example:  toggleLogsSelection

Where?: in the form: <%= form......   <% end %>


        <script language="javascript" type="text/javascript">

        function toggleLogsSelection(name) {
                var  boxes = document.getElementsByClassName(name);
                var all_checked = true;
                for (i = 0; i < boxes.length; i++) { if (boxes[i].checked == false) { all_checked = false; } }
                for (i = 0; i < boxes.length; i++) {
                        if (all_checked) {
                                boxes[i].checked = false;
                                boxes[i].up('tr').removeClassName('context-menu-selection');
                        } else if (boxes[i].checked == false) {
                                boxes[i].checked = true;
                                boxes[i].up('tr').addClassName('context-menu-selection');
                        }
                }
        }
        </script>


###3 Add selectall /unselectall link.

example: x-select

Where?:title bar: <thead>...</thead>

          <th scope="col"><%= link_to image_tag('toggle_check.png'), {}, :onclick => "toggleLogsSelection('logs-select'); return false;", :title => "check_all/uncheck_all" %></th>

### 
