import FloatButton from '@ifca-root/react-component/src/components/Button/FloatButton'
import MainHeader from '@ifca-root/react-component/src/components/Header/MainHeader'
import TextSeparator from '@ifca-root/react-component/src/components/Typography/TextSeparator'
import {
  IconButton,
  List,
  ListItem,
  ListItemSecondaryAction,
  ListItemText,
  Menu,
  MenuItem,
} from '@material-ui/core'
import { MoreVert } from '@material-ui/icons'
import { ContentWrapper } from 'components/Layout/ContentWrapper'
import { SearchBar } from '@ifca-root/react-component/src/components/SpecialBar/SearchBar'
import AppContext from 'containers/App/Store/AppContext'
import { useMenuOption } from 'helpers/CustomHooks/useMenuOption'
import React, { useContext, useEffect, useRef, useState } from 'react'
import { useHistory, useLocation, useParams } from 'react-router'
import { useGetAccountQuery } from 'generated/graphql'
import { useDateFormat } from 'helpers/CustomHooks/useDateFormat'
import { style } from '@material-ui/system'
import NumberFormat from 'react-number-format'

export const SubscribedProductList = (props: any) => {
  const { globalState, dispatch } = useContext(AppContext as any)
  let history = useHistory()
  let location = useLocation()
  const { convertDate } = useDateFormat()

  const subInfo: any = location?.state
  console.log('info', subInfo)
  const { anchorEl, menu, handleClick, handleClose } = useMenuOption()

  const { accountID } = useParams()
  console.log('ID', accountID)

  const {
    data: { getAccount } = { getAccount: [] },
    loading,
    error,
  } = useGetAccountQuery({ variables: { ID: accountID } })

  console.log('GETACCOUNT', getAccount)

  const activeSubs = getAccount[0]?.subscription?.filter(x => !x.isTerminated)

  return (
    <>
      <MainHeader
        mainBtn="back"
        onClick={() =>
          history.push({
            pathname: `/account/${accountID}/submenu`,
            state: getAccount[0],
          })
        }
        smTitle="Accounts"
        title={getAccount[0]?.name}
        routeSegments={[
          { name: 'Main' },
          { name: 'Accounts' },
          { name: 'Submenu' },
          { name: 'Subscribed Product', current: true },
        ]}
      />

      <List className="core-list subheader">
        <ListItem>
          <ListItemText
            primary={
              <>
                <span className="xsTitle highlight-text">
                  {getAccount[0]?.name}
                </span>
              </>
            }
            secondary={
              <span className="desc">
                <span className="text">Active Product: </span>
                <span className="highlight-text"> {activeSubs?.length}</span>
              </span>
            }
          />
        </ListItem>
      </List>
      <List>
        <SearchBar />
      </List>

      <ContentWrapper>
        <List className="core-list">
          {getAccount[0]?.subscription?.map((el, index) => (
            <ListItem
              key={index}
              selected={el.isTerminated}
              style={{
                textDecoration:
                  el.isTerminated === true ? 'line-through' : null,
              }}
              onClick={() =>
                history.push({
                  pathname: el.isTerminated
                    ? `/account/submenu/${accountID}/subscribed/`
                    : `/account/submenu/subscribed/${accountID}/view`,
                  state: {
                    data: getAccount[0],
                    index: index,
                  },
                })
              }
            >
              <ListItemText
                key={index}
                primary={
                  <>
                    <span
                      //aria-disabled={el.isTerminated}
                      className="xsTitle text flex-space"
                    >
                      {el?.software?.name}
                    </span>
                    <span className="desc color-orange">MYR</span>
                    <span className="desc highlight-text">
                      <NumberFormat
                        thousandSeparator
                        displayType="text"
                        type="text"
                        className="xxTitle highlight-text"
                        value={el?.subscriptionAmt}
                        isNumericString
                      />
                    </span>
                  </>
                }
                secondary={
                  <>
                    <span className="desc text ">Total Project: </span>
                    <span className="desc highlight-text"> 2</span>
                    <TextSeparator />
                    <span className="desc text ">Total User: </span>
                    <span className="desc highlight-text flex-space">
                      {' '}
                      {el.noOfUser}
                    </span>
                    <span className="desc">
                      {convertDate(el?.startDate)}
                      {'-'}
                      {convertDate(el?.endDate)}
                    </span>
                  </>
                }
              />
              <ListItemSecondaryAction>
                <IconButton
                  edge="end"
                  aria-label="delete"
                  onClick={e => handleClick(e, el.ID, index, el)}
                >
                  <MoreVert />
                </IconButton>
              </ListItemSecondaryAction>
            </ListItem>
          ))}
        </List>

        <Menu
          id="menu-list"
          anchorEl={anchorEl}
          keepMounted
          open={Boolean(anchorEl)}
          onClose={handleClose}
        >
          <MenuItem
            disabled={menu?.obj?.isTerminated}
            onClick={() =>
              history.push({
                pathname: `/account/submenu/subscribed/${accountID}/view`,
                state: {
                  data: getAccount[0],
                  index: menu?.index,
                },
              })
            }
          >
            <span className="">View</span>
          </MenuItem>

          <MenuItem
            disabled={menu?.obj?.isTerminated}
            onClick={() =>
              history.push({
                pathname: `/account/submenu/subscribed/${accountID}/edit/${menu.ID}`,
                state: {
                  data: getAccount[0],
                  index: menu?.index,
                },
              })
            }
          >
            <span className="">Edit</span>
          </MenuItem>
        </Menu>
      </ContentWrapper>
      <FloatButton
        onClick={() =>
          history.push({
            pathname: `/account/submenu/subscribed/${accountID}/add`,
            state: {
              data: getAccount[0],
              index: menu.index,
            },
          })
        }
      />
    </>
  )
}
