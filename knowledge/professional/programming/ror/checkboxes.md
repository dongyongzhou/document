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
    
        @xxx = InfoLog.find(params[:xxx_ids])
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

