# Details

```
Home Assistant (Raspberry Pi)
|_ scrapes Amazon/Target/... websites
|_ send a notification on state change to Gotify (Raspberry Pi)
   |_ Gotify app (Android) receives the notification and automatically opens the item page to buy
```

## Notes

- Things can probably run without public IP, but in case you want to reuse what I've done:
   - My Home Assistant and Gotify are running on 2 different domains - same public IP, because I like to keep things separate
   - Domains are reserved with NoIP
   - SSL Certificates are created with Let's Encrypt

## Home Assistant `configuration.yaml`

```
notify:
- name: gotify_iot_amazon_ps5
  platform: rest
  resource: https://<your gotify domain>/gotify/message
  method: POST_JSON
  headers:
    X-Gotify-Key: !secret gotify_iotwebthings_key
  message_param_name: message
  title_param_name: title
  data:
    priority: 10
    extras:
      client::notification:
        click:
          url: https://www.amazon.com.au/PlayStation-5-Console/dp/B08HHV8945
      android::action:
        onReceive:
          intentUrl: https://www.amazon.com.au/PlayStation-5-Console/dp/B08HHV8945
- name: gotify_iot_bigw_ps5
  platform: rest
  resource: https://<your gotify domain>/gotify/message
  method: POST_JSON
  headers:
    X-Gotify-Key: !secret gotify_iotwebthings_key
  message_param_name: message
  title_param_name: title
  data:
    priority: 10
    extras:
      client::notification:
        click:
          url: https://www.bigw.com.au/product/playstation-5-console/p/124625/
      android::action:
        onReceive:
          intentUrl: https://www.bigw.com.au/product/playstation-5-console/p/124625/
- name: gotify_iot_target_ps5_bundle
  platform: rest
  resource: https://<your gotify domain>/gotify/message
  method: POST_JSON
  headers:
    X-Gotify-Key: !secret gotify_iotwebthings_key
  message_param_name: message
  title_param_name: title
  data:
    priority: 10
    extras:
      client::notification:
        click:
          url: https://www.target.com.au/p/playstation-reg-5-console-with-additional-dualsense-wireless-controller-bundle/65618097
      android::action:
        onReceive:
          intentUrl: https://www.target.com.au/p/playstation-reg-5-console-with-additional-dualsense-wireless-controller-bundle/65618097
- name: gotify_iot_target_ps5_regular
  platform: rest
  resource: https://<your gotify domain>/gotify/message
  method: POST_JSON
  headers:
    X-Gotify-Key: !secret gotify_iotwebthings_key
  message_param_name: message
  title_param_name: title
  data:
    priority: 10
    extras:
      client::notification:
        click:
          url: https://www.target.com.au/p/playstation-reg-5-console/64226187
      android::action:
        onReceive:
          intentUrl: https://www.target.com.au/p/playstation-reg-5-console/64226187

multiscrape:
  Playstation 5 (PS5) - Amazon
  - resource: https://www.amazon.com.au/Sony-PlayStation-5-Console/dp/B08HHV8945
    headers:
      User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.54 Safari/537.36
    sensor:
      - unique_id: amazon_ps5_avail
        name: Amazon PS5 Availability
        select: "#availability > span"
        value_template: '{{ value|trim }}'
  Playstation 5 (PS5) - BigW
  - resource: https://www.bigw.com.au/product/playstation-5-console/p/124625/
    headers:
      User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.54 Safari/537.36
    sensor:
      - unique_id: bigw_ps5_avail
        name: BigW PS5 Availability
        select: "div.product-images > div[class^='ProductLabels_'] > div"
        value_template: '{{ value|trim }}'
        on_error:
          value: default
          default: not available
  Playstation 5 (PS5) - Target
  - resource: https://www.target.com.au/playstation-5
    headers:
      User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.54 Safari/537.36
    sensor:
      - unique_id: target_ps5_news
        name: Target PS5 News
        select: "div.ps5-container > div > p"
        value_template: '{{ value|trim }}'
        on_error:
          value: default
          default: not available
  - resource: https://www.target.com.au/p/playstation-reg-5-console-with-additional-dualsense-wireless-controller-bundle/65618097
    headers:
      User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.54 Safari/537.36
    sensor:
      - unique_id: target_ps5_avail_bundle
        name: Target PS5 Availability bundle
        select: "div.prod-basic > div.prod-heading > h1"
        value_template: '{{ value|trim }}'
        on_error:
          value: default
          default: not available
  - resource: https://www.target.com.au/p/playstation-reg-5-console/64226187
    headers:
      User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.54 Safari/537.36
    sensor:
      - unique_id: target_ps5_avail_regular
        name: Target PS5 Availability regular
        select: "div.prod-basic > div.prod-heading > h1"
        value_template: '{{ value|trim }}'
        on_error:
          value: default
          default: not available
```

## Home Assistant `secrets.yaml`

```
gotify_iotwebthings_key: <whatever key created in Gotify>
```

## Home Assistant `automations.yaml`

```
- id: '1632448249710'
  alias: Notify when PS5 is available (Amazon)
  description: ''
  trigger:
  - platform: state
    entity_id: sensor.amazon_ps5_avail
  condition:
  - condition: template
    value_template: '{{ trigger.to_state.state != trigger.from_state.state }}'
  action:
  - service: persistent_notification.create
    data:
      title: Amazon PS5
      message: PS5 is now {{ states('sensor.amazon_ps5_avail') }}
  - service: notify.gotify_iot_amazon_ps5
    data:
      title: Amazon PS5
      message: PS5 is now {{ states('sensor.amazon_ps5_avail') }}
  mode: single
- id: '1634519094313'
  alias: Notify when PS5 is available (BigW)
  description: ''
  trigger:
  - platform: state
    entity_id: sensor.bigw_ps5_avail
  condition:
  - condition: template
    value_template: '{{ trigger.to_state.state != trigger.from_state.state }}'
  action:
  - service: persistent_notification.create
    data:
      title: BigW PS5
      message: PS5 is now {{ states('sensor.bigw_ps5_avail') }}
  - service: notify.gotify_iot_bigw_ps5
    data:
      title: BigW PS5
      message: PS5 is now {{ states('sensor.bigw_ps5_avail') }}
  mode: single
- id: '1636114796139'
  alias: Notify when PS5 bundle is available (Target)
  description: ''
  trigger:
  - platform: state
    entity_id: sensor.target_ps5_avail_bundle
  condition:
  - condition: template
    value_template: '{{ trigger.to_state.state != trigger.from_state.state }}'
  action:
  - service: persistent_notification.create
    data:
      title: Target PS5 bundle
      message: PS5 is now {{ states('sensor.target_ps5_avail_bundle') }}
  - service: notify.gotify_iot_target_ps5_bundle
    data:
      title: Target PS5 bundle
      message: PS5 is now {{ states('sensor.target_ps5_avail_bundle') }}
  mode: single
- id: '1636114914473'
  alias: Notify when PS5 regular is available (Target)
  description: ''
  trigger:
  - platform: state
    entity_id: sensor.target_ps5_avail_regular
  condition:
  - condition: template
    value_template: '{{ trigger.to_state.state != trigger.from_state.state }}'
  action:
  - service: persistent_notification.create
    data:
      title: Target PS5 regular
      message: PS5 is now {{ states('sensor.target_ps5_avail_regular') }}
  - service: notify.gotify_iot_target_ps5_regular
    data:
      title: Target PS5 regular
      message: PS5 is now {{ states('sensor.target_ps5_avail_regular') }}
  mode: single
- id: '1636115219991'
  alias: Notify when PS5 news updated (Target)
  description: ''
  trigger:
  - platform: state
    entity_id: sensor.target_ps5_news
  condition:
  - condition: and
    conditions:
    - condition: template
      value_template: '{{ trigger.to_state.state != trigger.from_state.state }}'
    - condition: template
      value_template: '{{ trigger.to_state.state != "unavailable" }}'
  action:
  - service: notify.gotify_iot
    data:
      title: PS5 news changed at Target
      message: Status changed to {{ states('sensor.target_ps5_news') }} !
  mode: single
```
